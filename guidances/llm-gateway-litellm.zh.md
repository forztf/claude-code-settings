# 将 LLM 网关 (LiteLLM) 作为 Claude Code 的模型提供商

有关如何将 [LiteLLM](https://docs.litellm.ai/) 作为 Claude Code 的 LLM 网关连接的指导。

> 注意：LiteLLM 提供了 100+ LLM 的统一接口，包括通过 Anthropic、Bedrock 和 Vertex AI 的 Claude 模型。这允许您将 Claude Code 与 LiteLLM 支持的任何 LLM 提供商一起使用，同时保持完全兼容性。

## 1) 安装 Claude Code 和部署 LiteLLM 代理

```sh
npm install -g @anthropic-ai/claude-code
pip install -U 'litellm[proxy]'
```

## 2) 配置和启动 LiteLLM 代理

创建一个以 GitHub Copilot 为示例的 LiteLLM 配置文件 `litellm_config.yaml`：

```yaml
general_settings:
  master_key: sk-dummy
litellm_settings:
  drop_params: true
model_list:
- model_name: gpt-5.1-codex
  model_info:
    mode: responses
    supports_vision: true
  litellm_params:
    model: github_copilot/gpt-5.1-codex
    drop_params: true
    extra_headers:
      editor-version: "vscode/1.95.0"
      editor-plugin-version: "copilot-chat/0.26.7"
- model_name: gpt-5.1-codex-max
  model_info:
    mode: responses
    supports_vision: true
  litellm_params:
    model: github_copilot/gpt-5.1-codex-max
    drop_params: true
    extra_headers:
      editor-version: "vscode/1.95.0"
      editor-plugin-version: "copilot-chat/0.26.7"
- model_name: claude-opus-4.5
  litellm_params:
    model: github_copilot/claude-opus-4.5
    drop_params: true
    extra_headers:
      editor-version: "vscode/1.95.0"
      editor-plugin-version: "copilot-chat/0.26.7"
- model_name: "*"
  litellm_params:
    model: "github_copilot/*"
    extra_headers:
      editor-version: "vscode/1.95.0"
      editor-plugin-version: "copilot-chat/0.26.7"
```

启动 LiteLLM 代理：

```sh
litellm -c litellm_config.yaml
```

启动后，您将看到：

```sh
...
请访问 https://github.com/login/device 并输入代码 XXXX-XXXX 进行身份验证。
...
```

打开链接，登录并验证您的 GitHub Copilot 账户。

## 3) 创建具有以下内容的 Claude Code 配置文件 `~/.claude/settings.json`

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "http://localhost:4000",
    "ANTHROPIC_AUTH_TOKEN": "sk-dummy",
    "ANTHROPIC_MODEL": "claude-sonnet-4.5",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "gpt-5-mini",
    "DISABLE_NON_ESSENTIAL_MODEL_CALLS": "1",
    "DISABLE_TELEMETRY": "1",
    "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": "1"
  }
}
```

## 4) 运行 claude

打开另一个终端，然后随意运行 `claude`。请务必阅读其[最佳实践](https://www.anthropic.com/engineering/claude-code-best-practices)以充分利用其功能。

## 替代配置

### 直接使用环境变量

```sh
export ANTHROPIC_BASE_URL="http://localhost:4000"
export ANTHROPIC_AUTH_TOKEN="sk-dummy"
export ANTHROPIC_MODEL="claude-sonnet-4.5"
export ANTHROPIC_DEFAULT_HAIKU_MODEL="gpt-5-mini"
export DISABLE_TELEMETRY="1"
export DISABLE_NON_ESSENTIAL_MODEL_CALLS="1"
export CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC="1"

claude
```
