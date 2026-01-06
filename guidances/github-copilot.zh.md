# 将 GitHub Copilot 作为 Claude Code 的模型提供商

有关如何将 [GitHub Copilot](https://github.com/features/copilot) 作为 Claude Code 模型提供商连接的指导。

> 注意：调用 GitHub Copilot 并不违反其政策，因为根据[此处的文档](https://docs.github.com/en/copilot/how-tos/build-copilot-extensions/building-a-copilot-agent-for-your-copilot-extension/using-copilots-llm-for-your-agent)，这是官方支持的。实际上，已经有许多 AI 工具（例如 Aider 和 Cline VSCode 扩展）已支持 GitHub Copilot 作为 LLM 提供商之一。

## 1) 安装 Claude Code 和 Copilot API 代理

```sh
npm install -g copilot-api @anthropic-ai/claude-code
```

## 2) 启动 copilot-api 并验证 GitHub Copilot

```
$ copilot-api start --proxy-env
...
请访问 https://github.com/login/device 并输入代码 XXXX-XXXX 进行身份验证
...
```

成功后，您将看到模型列表和 API 地址：

```sh
...
- claude-3.5-sonnet
- claude-3.7-sonnet
- claude-3.7-sonnet-thought
- claude-sonnet-4.5
- claude-opus-4
- gemini-2.0-flash-001
- gemini-2.5-pro
- o3
...
  ➜ 监听地址：http://localhost:4141/ (所有接口)
```

## 3) 创建具有以下内容的 Claude Code 配置文件 `~/.claude/settings.json`

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "http://localhost:4141",
    "ANTHROPIC_AUTH_TOKEN": "sk-dummy",
    "ANTHROPIC_MODEL": "claude-sonnet-4.5",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "gpt-5-mini",
    "DISABLE_NON_ESSENTIAL_MODEL_CALLS": "1",
    "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": "1"
  }
}
```

## 4) 运行 claude

打开另一个终端，然后随意运行 `claude`。请务必阅读其[最佳实践](https://www.anthropic.com/engineering/claude-code-best-practices)以充分利用其功能。

## 替代配置

如果上述配置的文件不起作用，请直接使用环境变量：

```sh
export ANTHROPIC_BASE_URL="http://localhost:4141"
export ANTHROPIC_AUTH_TOKEN="sk-dummy"
export ANTHROPIC_MODEL="claude-sonnet-4.5"
export ANTHROPIC_DEFAULT_HAIKU_MODEL="gpt-5-mini"
export DISABLE_NON_ESSENTIAL_MODEL_CALLS="1"
export CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC="1"

claude
```
