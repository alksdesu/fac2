# Factory AI Proxy

一个 OpenAI 兼容的反向代理服务器，支持将请求转发到 Factory AI 和 Claude API

## 功能特性

- **多端点支持**
  - OpenAI 兼容格式端点：`/v1/chat/completions`
  - Claude 原生格式端点：`/v1/messages`

- **多模型支持**
  - Factory AI 模型（通过 OpenAI 端点）
  - Claude 系列模型（两个端点都支持）
  - Bedrock 模型（模型名包含 `bedrock` 前缀）
  - Vertex 模型（模型名包含 `vertex` 前缀）

- **Claude 特性**
  - 思考模式：模型名包含 `-thinking` 后缀时自动启用
  - 搜索模式：模型名包含 `-search` 后缀启用 web_search 工具
  - Opus 4.5：自动启用 effort=high 和扩展思考
  - 提示词缓存：自动添加缓存断点
  - 1M token context 支持
  - 流式响应支持
  - 多模态支持（文本和图片）
  - 工具调用支持（自动转换格式）

- **API Key 管理**
  - 多 Key 轮询
  - 401/402 错误自动禁用并切换到下一个 Key

## 安装

1. 安装 Bun
```bash
# Windows
powershell -c "irm bun.sh/install.ps1 | iex"

# macOS / Linux
curl -fsSL https://bun.sh/install | bash
```

2. 安装依赖
```bash
bun install
```

3. 配置环境变量
```bash
cp .env.example .env
```

## 配置说明

在 `.env` 文件中配置以下环境变量：

```env
# Factory AI API 密钥（多个用分号分隔）
FACTORY_API_KEYS=your_key1;your_key2;your_key3

# 代理访问密钥（可选，用于限制访问）
PROXY_ACCESS_KEYS=proxy_key1;proxy_key2

# 自定义代理密钥请求头（默认：X-Proxy-Key）
PROXY_KEY_HEADER=X-Proxy-Key

# 代理端口（默认：8001）
PROXY_PORT=8001
```

## 运行

```bash
bun run main.ts
```

## API 使用

### OpenAI 兼容格式端点

**端点**: `POST http://localhost:8001/v1/chat/completions`

**请求示例**:
```json
{
  "model": "claude-sonnet-4-5-thinking",
  "messages": [
    {"role": "user", "content": "Hello!"}
  ],
  "stream": true,
  "max_tokens": 4096
}
```

**模型名示例**:
- `claude-sonnet-4-5` - 标准模式
- `claude-sonnet-4-5-thinking` - 启用思考模式
- `claude-opus-4-5-search` - 启用搜索工具
- `bedrock-claude-3-5-sonnet-20241022` - Bedrock 模型
- `vertex-claude-3-5-sonnet-20241022` - Vertex 模型

### Claude 原生格式端点

**端点**: `POST http://localhost:8001/v1/messages`

直接支持 Claude 原生请求格式，保留所有原始功能。

## 认证方式

1. **使用自己的 API Key**
   ```
   Authorization: Bearer your-api-key
   ```

2. **使用配置的 Factory API Keys**
   - 不提供 Authorization header，系统会自动轮询使用配置的密钥

3. **使用代理访问密钥**（如果配置了 PROXY_ACCESS_KEYS）
   ```
   X-Proxy-Key: your-proxy-key
   ```
