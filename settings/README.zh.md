# 设置

此目录包含针对不同设置的 Claude Code 设置配置示例。

所有配置包括：

- 不同提供商的自定义基础 URL 或 API 端点
- 身份验证令牌占位符
- 每个提供商的模型规格
- 为开发禁用遥测和非必要流量

## 可用设置

### [copilot-settings.json](copilot-settings.json)

使用 GitHub Copilot 代理设置配置 Claude Code。指向 localhost:4141 作为 Anthropic API 基础 URL。

### [litellm-settings.json](litellm-settings.json)

使用 LiteLLM 网关设置配置 Claude Code。指向 localhost:4000 作为 Anthropic API 基础 URL。

### [qwen-settings.json](qwen-settings.json)

使用阿里巴巴 DashScope API 的 Qwen 模型配置 Claude Code。通过 claude-code-proxy 使用 Qwen3-Coder-Plus 模型。

### [siliconflow-settings.json](siliconflow-settings.json)

使用 SiliconFlow API 配置 Claude Code。使用 Moonshot AI Kimi-K2-Instruct 模型。

### [vertex-settings.json](vertex-settings.json)

使用 Google Cloud Vertex AI 配置 Claude Code。使用 Claude Opus 4 模型和 Google Cloud 项目设置。

### [deepseek-settings.json](deepseek-settings.json)

使用 DeepSeek v3.1（通过 DeepSeek 的官方 Anthropic 兼容 API）配置 Claude Code。

### [azure-settings.json](azure-settings.json)

使用 Azure AI（Anthropic 兼容端点）配置 Claude Code。指向 Azure AI 服务端点。

### [azure-foundry-settings.json](azure-foundry-settings.json)

使用 Azure AI Foundry 本机模式配置 Claude Code。使用 `CLAUDE_CODE_USE_FOUNDRY` 标志和 Claude Opus 4.1 + Sonnet 4.5 模型。

### [minimax.json](minimax.json)

使用 MiniMax API 配置 Claude Code。使用 MiniMax-M2 模型。

### [openrouter-settings.json](openrouter-settings.json)

使用 OpenRouter API 配置 Claude Code。OpenRouter 通过统一 API 访问许多模型。注意：`ANTHROPIC_API_KEY` 必须明确为空，而 `ANTHROPIC_AUTH_TOKEN` 包含您的 OpenRouter API 密钥。默认情况下，Claude Code 使用 Anthropic 模型别名（Sonnet、Opus、Haiku），OpenRouter 会自动映射这些别名。您可以使用 `ANTHROPIC_DEFAULT_SONNET_MODEL`、`ANTHROPIC_DEFAULT_OPUS_MODEL` 和 `ANTHROPIC_DEFAULT_HAIKU_MODEL` 覆盖自定义模型。
