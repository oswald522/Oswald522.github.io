---
title: "Alpine搭建K8s教程"
description: "基于 Alpine 使用 kubeadm 搭建 k8s"
date: 2025-04-22T15:13:34+08:00
Lastmod: 2025-04-22T15:13:34+08:00
draft: false
showComments: true
featureimage: "https://picsum.photos/seed/fccce8/1600/900.webp"
tags:
  - 操作系统
  - 服务器
  - 技术教程
series: ["Alpine操作系统"]
series_order: 2
---


## 基于 Alpine 使用 kubeadm 搭建 k8s

先部署[基础环境](https://www.cnblogs.com/chongxs/p/17982220/alpine-base-env)，然后根据官方文档 [K8s - Alpine Linux](https://wiki.alpinelinux.org/wiki/K8s)，进行操作。

### 将官方文档整理为脚本

整理脚本时，有部分调整

```bash
#!/bin/sh

set -x
# 添加源，安装时已经配置
#cat >> /etc/apk/repositories <<"EOF"
#http://mirrors.aliyun.com/alpine/edge/community
#http://mirrors.aliyun.com/alpine/edge/testing
#EOF

# 加载模块
echo "br_netfilter" > /etc/modules-load.d/k8s.conf
modprobe br_netfilter
# 临时加载，改为写入文件，防止重启失效
#echo 1 > /proc/sys/net/ipv4/ip_forward

apk add cni-plugin-flannel
apk add flannel
apk add flannel-contrib-cni
apk add cni-plugins
apk add kubelet
apk add kubeadm
apk add kubectl
apk add uuidgen
apk add nfs-utils
apk add containerd
# 把管理工具 ctr 安装上
apk add containerd-ctr
#apk add bash
#apk add vim

# 关闭 swap
swapoff -a
sed -i "s|.*swap.*|#&|" /etc/fstab

#Fix prometheus errors
mount --make-rshared /
cat > /etc/local.d/sharemetrics.start <<"EOF"
#!/bin/sh
mount --make-rshared /
EOF

chmod +x /etc/local.d/sharemetrics.start
rc-update add local

#Fix id error messages
uuidgen > /etc/machine-id

#Add services
#rc-update add docker
rc-update add containerd
rc-update add kubelet
rc-service containerd start

#kernel stuff
cat >> /etc/sysctl.conf <<"EOF"
net.bridge.bridge-nf-call-iptables=1
net.ipv4.ip_forward= 1
EOF
# set net work
sysctl -p
```

#### 修改 containerd 配置

- 查看当前初始化使用的镜像信息

    ```shell
    kubeadm config images list
    ```

- 修改镜像加速 `/etc/containerd/config.toml`

    ```toml
    [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
              [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
                endpoint = ["https://kuamavit.mirror.aliyuncs.com", "https://registry-1.docker.io"]
    ```

- 修改sandbox `/etc/containerd/config.toml`

    ```toml
    sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.9"
    ```

- 改为配置，重启服务 `rc-service containerd restart`

___

- 可基于以上基础环境克隆设备
- 设备需要修改静态IP，修改 hostname，修改 hosts 文件

___

#### 初始化【此前操作，所有节点均需要】

- 尝试初始化，拉取镜像超级慢，建议先拉取镜像

    ```shell
    kubeadm init --pod-network-cidr=10.244.0.0/16 --node-name=$(hostname) --image-repository registry.aliyuncs.com/google_containers
    ```

- 先拉取镜像

    ```sh
    kubeadm config images pull --image-repository registry.aliyuncs.com/google_containers
    ```

    ```sh
    kubeadm init \
    --node-name=$(hostname) \
    --apiserver-advertise-address=172.16.14.201 \
    --image-repository registry.aliyuncs.com/google_containers \
    --service-cidr=10.96.0.0/12 \
    --pod-network-cidr=10.244.0.0/16 --v=5
    ```

- 根据提示，配置环境变量

    ```sh
    mkdir -p $HOME/.kube
    cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    chown $(id -u):$(id -g) $HOME/.kube/config
    export KUBECONFIG=/etc/kubernetes/admin.conf
    ```

- 查看状态，因为没安装网络插件，所以是没启动成功

    ```bash
    # 查看状态，NotReady
    kubectl get nodes
    # 查看pod，coredns 在 pending 状态
    kubectl get pod -A
    ```

#### 安装网络组件

##### 待处理方案，coredns 加载不上【calico、cilium均有相似问题，待处理】

- 利用 helm 安装，添加仓库

    ```bash
    # helm repo add projectcalico https://docs.tigera.io/calico/charts
    helm repo add cilium https://helm.cilium.io/
    ```

- 安装组件，calico 不好拉，cilium 要更容易拉取

    ```bash
    helm install cilium cilium/cilium --version 1.11.20  --namespace kube-system
    ```

- 观察一手

    ```bash
    watch kubectl get pods -A -o wide
    ```

##### 【可用方案】使用 flannel

- 拉取 flannel yaml 文件

```shell
wget -c  https://mirror.ghproxy.com/https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

- 拉一把试试

```shell
kubectl apply -f kube-flannel.yml
# 镜像有点难拉，单独拉
ctr -n k8s.io i pull docker.io/flannel/flannel:v0.24.2
# docker.io/flannel/flannel-cni-plugin:v1.4.0-flannel1 这个镜像好像比较好拉
```

#### 节点加入

- 打印加入 token

```shell
kubeadm token create --print-join-command
```
