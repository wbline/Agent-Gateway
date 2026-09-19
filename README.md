# Agent Gateway - 智能体网关

一个本地网关，统一管理多家大模型供应商，让 AI Agent 无缝调用所有模型

## 🌟 项目简介

Agent Gateway 是一个**本地部署**的智能体网关，为 AI 代理提供**统一的模型调用入口**。

它将多家大模型供应商（OpenAI、Anthropic、Google、DeepSeek 等）的 API 聚合到一个本地端点，让您的 Agent **只需对接一个 OpenAI 兼容接口**，即可无缝调用所有支持的模型。



## ✨ 核心特性

### 🎯 统一网关入口
| 特性 | 说明 |
|------|------|
| **多协议支持** | OpenAI Completions / Responses、Anthropic Messages、Google Gemini 四种协议 |
| **智能路由** | 基于模型 ID 前缀自动路由到对应供应商 |
| **协议透明** | 对外协议与上游一致，不做协议间转换 |

### 🤖 智能模型路由（Auto Model）
| 策略 | 说明 |
|------|------|
| **轮询** | 依次使用候选池中的模型 |
| **随机** | 随机选择一个可用模型 |
| **加权** | 按权重概率选择模型 |
| **级联降级** | 优先使用高优先级模型，失败时降级 |

### ⚡ 流量控制与保护
- **供应商限流**：按供应商配置 RPM，自动控制请求速率
- **智能排队**：超出限流的请求自动排队，客户端不断开
- **故障自愈**：自动重试或切换到下一个可用模型

### 🔐 安全与隐私
- **100% 本地部署**：所有数据存储在本地，不上传任何第三方
- **密钥加密**：API Key 使用本地加密密钥安全存储
- **单实例保护**：防止重复启动

### 📊 实时监控
- **WebSocket 推送**：实时查看活跃请求、Token 速率
- **多维度统计**：按天、小时、分钟粒度的使用分析
- **请求明细**：完整的请求记录，支持筛选



## 📖 使用指南

### 1️⃣ 添加供应商

1. 打开应用，点击左侧菜单「🔑 供应商管理」
2. 点击「添加供应商」
3. 填写信息：
   - **名称**：如 `OpenAI`、`DeepSeek`
   - **Base URL**：API 端点地址
   - **API Key**：供应商的密钥
4. 点击「测试连接」验证
5. 点击「保存」

### 2️⃣ 启动网关

1. 点击左侧菜单「🚀 网关」
2. 配置服务：
   - **监听端口**：默认 `3001`
   - **认证令牌**：客户端连接使用的 Token
   - **对外协议**：选择对外提供的 API 协议
3. 点击「启动服务」

### 3️⃣ 接入 Agent

**Python (OpenAI SDK)**：

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://127.0.0.1:3001/v1",
    api_key="your-gateway-token"
)

response = client.chat.completions.create(
    model="deepseek:deepseek-r1",  # 格式：供应商ID:模型名
    messages=[{"role": "user", "content": "你好"}]
)

print(response.choices[0].message.content)
```

**curl**：

```bash
curl http://127.0.0.1:3001/v1/chat/completions \
  -H "Authorization: Bearer your-gateway-token" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "deepseek:deepseek-r1",
    "messages": [{"role": "user", "content": "你好"}]
  }'
```

### 4️⃣ 使用 Auto Model

不确定用哪个模型？使用 `auto` 让网关自动选择：

```python
response = client.chat.completions.create(
    model="auto",  # 智能路由
    messages=[{"role": "user", "content": "写一首诗"}]
)
```
