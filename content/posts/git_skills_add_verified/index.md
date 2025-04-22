---
title: "GitHub技巧及教程"
description: "GitHub添加验证(Verified)标识符等"
date: 2021-03-23T10:30:28+08:00
draft: false
showComments: true
featureimage: "https://picsum.photos/seed/8ee15e/1600/900.webp"
tags: ["GitHub","软件技巧"]
# series: "Git使用技巧"
---

## 前言

GitHub 是一个广泛使用的代码托管平台，除了基本的代码管理功能外，它还提供了许多高级功能来增强用户体验和安全性。其中之一就是 Verified（已验证） 标识符。当你的 commit 或 tag 被标记为 "Verified" 时，表示该提交或标签是通过 GPG 或 SSH 签名的，确保其真实性和完整性。本文将详细介绍如何为你的 GitHub 提交添加 Verified 标识符。

---

## 什么是 Verified 标识符？

Verified 标识符是 GitHub 对 GPG 或 SSH 签名提交的一种验证标记。它表明：

- 提交的作者身份是可信的。
- 提交内容未被篡改。

这对于开源项目或团队协作非常重要，因为它可以防止恶意提交或伪造提交。

---

## 添加 Verified 标识符的步骤

### 方法 1：使用 GPG 签名

#### 1. 安装 GPG

如果你的系统尚未安装 GPG，请先安装：

  ```bash
brew install gnupg # macOS
sudo apt-get install gnupg  # Linux
# Windows: 下载并安装 [Gpg4win](https://www.gpg4win.org/)。
```

#### 2. 生成 GPG 密钥

运行以下命令`gpg --full-generate-key`生成 GPG 密钥，按照提示选择密钥类型、密钥长度（推荐 4096），并填写你的姓名和邮箱地址。

- 默认密钥类型为ECC（签名和加密），Curve 25519 ，永不过期
- 姓名建议直接使用Github用户名
- 邮箱地址选择为Github提供的隐私地址，通常为`随机字符+用户名@users.noreply.github.com.`，可以访问[Github邮箱](https://github.com/settings/emails)

#### 3. 查看 GPG 密钥

生成密钥后，运行以下命令`gpg --list-secret-keys --keyid-format LONG`查看，复制 `sec` 行中的密钥 ID（例如：`3AA5C34371567BD2`）。

#### 4. 导出 GPG 公钥

运行以下命令`gpg --armor --export YOUR_KEY_ID`导出公钥，将输出内容复制到剪贴板，需要包含`--Begin XXXX---`。

#### 5. 将 GPG 公钥添加到 GitHub

1. 登录 GitHub。
2. 访问 [GitHub SSH and GPG Keys 页面](https://github.com/settings/keys)。
3. 点击 New GPG key，将复制的公钥粘贴到输入框中，然后点击 Add GPG key。

#### 6. 配置 Git 使用 GPG 签名

运行以下命令配置 Git：

```bash
git config --global user.signingkey YOUR_KEY_ID
git config --global commit.gpgsign true
```

#### 7. 提交代码

现在，每次提交代码时，Git 会自动使用 GPG 签名。推送代码后，在 GitHub 上查看提交记录，你会看到 Verified 标识符。

---

### 方法 2：使用 SSH 签名

#### 1. 生成 SSH 密钥

如果尚未生成 SSH 密钥，运行以下命令：

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

#### 2. 将 SSH 公钥添加到 GitHub

1. 复制公钥内容：

   ```bash
   cat ~/.ssh/id_ed25519.pub
   ```

2. 登录 GitHub，访问 [GitHub SSH and GPG Keys 页面](https://github.com/settings/keys)。
3. 点击 New SSH key，将复制的公钥粘贴到输入框中，然后点击 Add SSH key。

#### 3. 配置 Git 使用 SSH 签名

运行以下命令：

```bash
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true
```

#### 4. 提交代码

提交代码后，GitHub 会自动验证 SSH 签名，并显示 Verified 标识符。

---

## 验证是否成功

提交代码后，访问 GitHub 仓库的提交记录页面。如果看到提交旁边有 Verified 标识符，说明配置成功。

---

## 常见问题

### 1. 为什么我的提交没有显示 Verified？

- 确保已正确配置 GPG 或 SSH 签名。
- 确保提交时使用了与 GPG 或 SSH 密钥关联的邮箱地址。

### 2. 如何更改 Git 的邮箱地址？

运行以下命令：

```bash
git config --global user.email "your_email@example.com"
```

### 3. 如何禁用 GPG 签名？

运行以下命令：

```bash
git config --global commit.gpgsign false
```

---

## 总结

通过 GPG 或 SSH 签名，可以为你的 GitHub 提交添加 Verified 标识符，增强提交的可信度和安全性。无论是个人项目还是团队协作，这一功能都非常有用。希望本文对你有所帮助！如果还有其他问题，欢迎留言讨论。
