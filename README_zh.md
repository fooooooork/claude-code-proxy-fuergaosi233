# Claude Code 代理

一个代理服务器，使 **Claude Code** 能够与 OpenAI 兼容的 API 提供商一起工作。将 Claude API 请求转换为 OpenAI API 调用，让您可以通过 Claude Code CLI 使用各种 LLM 提供商。

![Claude Code 代理](demo.png)

## 功能特性

- **完整的 Claude API 兼容性**：完整的 `/v1/messages` 端点支持
- **多提供商支持**：OpenAI、Azure OpenAI、本地模型（Ollama）和任何 OpenAI 兼容的 API
- **智能模型映射**：通过环境变量配置 BIG 和 SMALL 模型
- **函数调用**：完整的工具使用支持，具有适当的转换
- **流式响应**：实时 SSE 流式支持
- **图像支持**：Base64 编码的图像输入
- **错误处理**：全面的错误处理和日志记录

## 快速开始

### 1. 安装依赖

```bash
# 使用 UV（推荐）
uv sync

# 或使用 pip
pip install -r requirements.txt
```

### 2. 配置

```bash
cp .env.example .env
# 编辑 .env 并添加您的 API 配置
# 注意：环境变量会自动从 .env 文件加载
```

### 3. 启动服务器

```bash
# 直接运行
python start_proxy.py

# 或使用 UV
uv run claude-code-proxy

# 或使用 docker compose
docker compose up -d
```

### 4. 与 Claude Code 一起使用

```bash
# 如果代理中未设置 ANTHROPIC_API_KEY：
ANTHROPIC_BASE_URL=http://localhost:8082 ANTHROPIC_API_KEY="any-value" claude

# 如果代理中设置了 ANTHROPIC_API_KEY：
ANTHROPIC_BASE_URL=http://localhost:8082 ANTHROPIC_API_KEY="exact-matching-key" claude
```

## 配置

应用程序使用 `python-dotenv` 自动从项目根目录的 `.env` 文件加载环境变量。您也可以在 shell 中直接设置环境变量。

### 环境变量

**必需：**

- `OPENAI_API_KEY` - 目标提供商的 API 密钥

**安全：**

- `ANTHROPIC_API_KEY` - 用于客户端验证的预期 Anthropic API 密钥
  - 如果设置，客户端必须提供此确切的 API 密钥才能访问代理
  - 如果未设置，将接受任何 API 密钥

**模型配置：**

- `BIG_MODEL` - Claude opus 请求的模型（默认：`gpt-4o`）
- `MIDDLE_MODEL` - Claude opus 请求的模型（默认：`gpt-4o`）
- `SMALL_MODEL` - Claude haiku 请求的模型（默认：`gpt-4o-mini`）

**API 配置：**

- `OPENAI_BASE_URL` - API 基础 URL（默认：`https://api.openai.com/v1`）

**服务器设置：**

- `HOST` - 服务器主机（默认：`0.0.0.0`）
- `PORT` - 服务器端口（默认：`8082`）
- `LOG_LEVEL` - 日志级别（默认：`WARNING`）

**性能：**

- `MAX_TOKENS_LIMIT` - 令牌限制（默认：`4096`）
- `REQUEST_TIMEOUT` - 请求超时（秒）（默认：`90`）

### 模型映射

代理将 Claude 模型请求映射到您配置的模型：

| Claude 请求                 | 映射到     | 环境变量   |
| ------------------------------ | ------------- | ---------------------- |
| 包含 "haiku" 的模型            | `SMALL_MODEL` | 默认：`gpt-4o-mini` |
| 包含 "sonnet" 的模型           | `MIDDLE_MODEL`| 默认：`BIG_MODEL`   |
| 包含 "opus" 的模型             | `BIG_MODEL`   | 默认：`gpt-4o`      |

### 提供商示例

#### OpenAI

```bash
OPENAI_API_KEY="sk-your-openai-key"
OPENAI_BASE_URL="https://api.openai.com/v1"
BIG_MODEL="gpt-4o"
MIDDLE_MODEL="gpt-4o"
SMALL_MODEL="gpt-4o-mini"
```

#### Azure OpenAI

```bash
OPENAI_API_KEY="your-azure-key"
OPENAI_BASE_URL="https://your-resource.openai.azure.com/openai/deployments/your-deployment"
BIG_MODEL="gpt-4"
MIDDLE_MODEL="gpt-4"
SMALL_MODEL="gpt-35-turbo"
```

#### 本地模型（Ollama）

```bash
OPENAI_API_KEY="dummy-key"  # 必需但可以是虚拟的
OPENAI_BASE_URL="http://localhost:11434/v1"
BIG_MODEL="llama3.1:70b"
MIDDLE_MODEL="llama3.1:70b"
SMALL_MODEL="llama3.1:8b"
```

#### 其他提供商

通过设置适当的 `OPENAI_BASE_URL`，可以使用任何 OpenAI 兼容的 API。

## 使用示例

### 基本聊天

```python
import httpx

response = httpx.post(
    "http://localhost:8082/v1/messages",
    json={
        "model": "claude-3-5-sonnet-20241022",  # 映射到 MIDDLE_MODEL
        "max_tokens": 100,
        "messages": [
            {"role": "user", "content": "Hello!"}
        ]
    }
)
```

## 与 Claude Code 集成

此代理设计为与 Claude Code CLI 无缝协作：

```bash
# 启动代理
python start_proxy.py

# 使用代理的 Claude Code
ANTHROPIC_BASE_URL=http://localhost:8082 claude

# 或永久设置
export ANTHROPIC_BASE_URL=http://localhost:8082
claude
```

## 测试

测试代理功能：

```bash
# 运行综合测试
python src/test_claude_to_openai.py
```

## 开发

### 使用 UV

```bash
# 安装依赖
uv sync

# 运行服务器
uv run claude-code-proxy

# 格式化代码
uv run black src/
uv run isort src/

# 类型检查
uv run mypy src/
```

### 项目结构

```
claude-code-proxy/
├── src/
│   ├── main.py  # 主服务器
│   ├── test_claude_to_openai.py    # 测试
│   └── [其他模块...]
├── start_proxy.py                  # 启动脚本
├── .env.example                    # 配置模板
└── README.md                       # 此文件
```

## 性能

- **异步/等待** 实现高并发
- **连接池** 提高效率
- **流式支持** 实现实时响应
- **可配置的超时** 和重试
- **智能错误处理** 具有详细的日志记录

## 许可证

MIT 许可证


