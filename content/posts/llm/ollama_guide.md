---
title: "Ollama本地部署大模型完全指南"
description: "本地部署ollama大模型，包含模型下载、程序安装及常见问题解答"
date: 2024-04-20T10:21:19+08:00
Lastmod: 2024-04-20T10:21:19+08:00
draft: false
showComments: true
featureimage: "https://picsum.photos/seed/82eb89/1600/900.webp"
tags: ["技术教程","经验转载","大模型部署"]
series: "大模型部署"
series_order: 2
---

## 🚀 Ollama 安装部署

 *本地运行大语言模型的终极解决方案*

### ⚙️ 系统要求

| 操作系统 | 支持状态 | 备注 |
|----------|----------|------|
| macOS    | ✅ 完整支持 | Intel/Apple Silicon |
| Windows  | ✅ 预览版 | 需要Windows 10+ |
| Linux    | ✅ 完整支持 | 主流发行版 |

### 📦 安装方法

#### 🍎 macOS 和 Windows

```bash
# 官方安装方式
访问 https://ollama.com 下载安装包
```

#### 🐧 Linux

```bash
# 自动安装
curl -fsSL https://ollama.com/install.sh | sh
# 或手动安装
sudo curl -L https://ollama.com/download/ollama-linux-amd64 -o /usr/bin/ollama
sudo chmod +x /usr/bin/ollama
```

#### 🐋 Docker

```bash
docker pull ollama/ollama
docker run -d -p 11434:11434 --name ollama ollama/ollama
```

## 🏁 快速入门

### 🔥 运行第一个模型

```bash
ollama run llama2
```

### 📚 模型库

访问官方模型库：<https://ollama.com/library>

| 模型 | 参数 | 大小 | 命令 |
|------|------|------|------|
| Llama 2 | 7B | 3.8GB | `ollama run llama2` |
| Mistral | 7B | 4.1GB | `ollama run mistral` |
| CodeLlama | 7B | 3.8GB | `ollama run codellama` |

## 🛠️ 进阶使用

### ✨ 自定义模型

#### 从GGUF导入

1. 创建Modelfile：

```dockerfile
FROM ./vicuna-33b.Q4_0.gguf
```

2. 创建并运行模型：

```bash
ollama create example -f Modelfile
ollama run example
```

#### 🎭 角色定制

示例：创建马里奥角色模型

```dockerfile
FROM llama2
PARAMETER temperature 1
PARAMETER num_ctx 4096
SYSTEM """
你是超级马里奥兄弟中的马里奥，只能以马里奥的身份回答。
"""
```

## 🔌 API 使用

### 🔄 REST API

```bash
# 生成响应
curl http://localhost:11434/api/generate -d '{
  "model": "llama2",
  "prompt":"为什么天空是蓝色的？"
}'
# 聊天API
curl http://localhost:11434/api/chat -d '{
  "model": "mistral",
  "messages": [
    { "role": "user", "content": "为什么天空是蓝色的？" }
  ]
}'
```

### 🤝 OpenAI兼容API

```python
from openai import OpenAI
client = OpenAI(
    base_url='http://localhost:11434/v1/',
    api_key='ollama',  # 必需的占位符
)
response = client.chat.completions.create(
    messages=[{"role": "user", "content": "你好！"}],
    model="llama2"
)
```

## 🧩 模型导入

### 🔄 从不同格式导入

#### GGUF格式

```bash
# 1. 创建Modelfile
echo 'FROM ./mistral-7b.Q4_0.gguf' > Modelfile
# 2. 创建模型
ollama create mymodel -f Modelfile
# 3. 运行测试
ollama run mymodel "你好吗？"
```

#### PyTorch/Safetensors

```bash
# 转换流程
git clone https://github.com/ollama/ollama
cd ollama
make -C llm/llama.cpp quantize
# 转换模型
python llm/llama.cpp/convert.py ./model --outfile converted.bin
llm/llama.cpp/quantize converted.bin quantized.bin q4_0
```

## ⚡ 性能优化

### 🎛️ 参数调整

| 参数 | 描述 | 建议值 |
|------|------|--------|
| `num_ctx` | 上下文窗口大小 | 2048-4096 |
| `temperature` | 创意性控制 | 0.7-1.3 |
| `top_p` | 核采样 | 0.7-0.95 |
| `num_gpu` | GPU层数 | 根据VRAM调整 |

### 🖥️ 硬件加速

```bash
# 强制使用特定加速库
OLLAMA_LLM_LIBRARY="cuda_v11" ollama serve
# AMD GPU支持
export HSA_OVERRIDE_GFX_VERSION="10.3.0"
```

## 🐞 故障排除

### 📋 日志位置

| 系统 | 路径 |
|------|------|
| macOS | `~/.ollama/logs/server.log` |
| Linux | `journalctl -u ollama` |
| Windows | `%LOCALAPPDATA%\Ollama` |

### 🔍 常见问题

```bash
# GPU不工作？
OLLAMA_LLM_LIBRARY="cpu_avx2" ollama serve
# 网络问题？
export OLLAMA_HOST="0.0.0.0"
```

## ❓ FAQ

### 🔄 如何更新？

```bash
# Linux/macOS
curl -fsSL https://ollama.com/install.sh | sh
# Windows自动更新或重新下载安装包
```

### 💾 模型存储位置

| 系统 | 路径 |
|------|------|
| macOS | `~/.ollama/models` |
| Linux | `/usr/share/ollama/models` |
| Windows | `C:\Users\<user>\.ollama\models` |

### 🔌 网络配置

```bash
# 公开Ollama服务
export OLLAMA_HOST="0.0.0.0"
# 通过Nginx代理
location / {
    proxy_pass http://localhost:11434;
}
```
