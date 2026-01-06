# Claude Code 设置/命令/技能集合 - 用于 Vibe Coding

精心策划的 Claude Code 设置、自定义命令、技能和子代理集合，专为增强开发工作流程而设计。此设置包括用于功能开发（规格驱动工作流）、代码分析、GitHub 集成和知识管理的专用命令、技能和子代理。

> 如需 OpenAI Codex 设置、配置和自定义提示，请参考 [feiskyer/codex-settings](https://github.com/feiskyer/codex-settings)。

## 设置

### 使用 Claude Code 插件

```sh
/plugin marketplace add feiskyer/claude-code-settings

# 安装主插件（命令、代理和技能）
/plugin install claude-code-settings

# 或者，单独安装技能而不包含命令/代理
/plugin install codex-skill               # Codex 自动化
/plugin install autonomous-skill          # 长时间任务自动化
/plugin install nanobanana-skill          # 图像生成
/plugin install kiro-skill                # Kiro 工作流
/plugin install spec-kit-skill            # Spec-Kit 工作流
/plugin install youtube-transcribe-skill  # YouTube 字幕提取
```

或者，通过 [Claude Plugins CLI](https://claude-plugins.dev) 运行一键安装，跳过市场设置：

```bash
npx claude-plugins install @feiskyer/claude-code-settings/claude-code-settings
```

这将自动添加市场并一步安装插件。

**注意：**

- [~/.claude/settings.json](settings.json) 不通过 Claude Code 插件配置，您需要手动配置。

### 手动设置

```sh
# 备份原始 claude 设置
mv ~/.claude ~/.claude.bak

# 克隆 claude-code-settings
git clone https://github.com/feiskyer/claude-code-settings.git ~/.claude

# 安装 LiteLLM 代理
pip install -U 'litellm[proxy]'

# 启动 litellm 代理（将监听 http://0.0.0.0:4000）
litellm -c ~/.claude/guidances/litellm_config.yaml

# 为方便起见，使用 tmux 在后台运行 litellm 代理
# tmux new-session -d -s copilot 'litellm -c guidances/litellm_config.yaml'
```

启动后，您将看到：

```sh
...
请访问 https://github.com/login/device 并输入代码 XXXX-XXXX 进行身份验证。
...
```

打开链接，登录并验证您的 GitHub Copilot 账户。

**注意：**

1. 默认配置利用 [LiteLLM 代理服务器](https://docs.litellm.ai/docs/simple_proxy)作为 GitHub Copilot 的 LLM 网关。您也可以使用 [copilot-api](https://github.com/ericc-ch/copilot-api) 作为代理（记得将端口更改为 4141）。
2. 确保您的账户中有以下可用模型；如果没有，请用您自己的模型名称替换：

   - ANTHROPIC_DEFAULT_SONNET_MODEL: claude-sonnet-4.5

   - ANTHROPIC_DEFAULT_OPUS_MODEL: claude-opus-4

   - ANTHROPIC_DEFAULT_HAIKU_MODEL: gpt-5-mini


## 命令

`commands/` 目录包含扩展 Claude Code 斜杠命令的[自定义斜杠命令](https://code.claude.com/docs/en/slash-commands)，可通过 `/<命令名称> [参数]` 调用。

<details>
<summary>分析与反思</summary>

### 分析与反思

- `/think-harder [问题]` - 增强分析思维
- `/think-ultra [复杂问题]` - 超全面分析
- `/reflection` - 分析和改进 Claude Code 指令
- `/reflection-harder` - 全面的会话分析和学习
- `/eureka [突破]` - 记录技术突破

</details>

<details>
<summary>GitHub 集成</summary>

### GitHub 集成

- `/gh:review-pr [PR编号]` - 全面的 PR 审查和评论
- `/gh:fix-issue [问题编号]` - 完整的问题解决工作流

</details>

<details>
<summary>文档与知识</summary>

### 文档与知识

- `/cc:create-command [名称] [描述]` - 创建新的 Claude Code 命令

</details>

<details>
<summary>工具</summary>

### 工具

- `/translate [文本]` - 将英语/日语技术内容翻译为中文

</details>

## 技能

技能现在作为单独的插件分发，以便模块化安装。仅安装您需要的内容：

<details>
<summary>codex-skill - 将任务移交给 Codex CLI</summary>

### [codex-skill](plugins/codex-skill)

使用 OpenAI Codex 进行免提任务执行的非交互式自动化模式。当您想要利用 codex、gpt-5 或 gpt-5.1 来实现 Claude 设计的功能或计划时使用。

**安装：**

```sh
/plugin marketplace add feiskyer/claude-code-settings
/plugin install codex-skill
```

**主要功能：**

- 多种执行模式（只读、工作区写入、危险完全访问）
- 模型选择支持（gpt-5、gpt-5.1、gpt-5.1-codex 等）
- 无需批准提示的自主执行
- 结构化结果的 JSON 输出支持
- 可恢复的会话

**要求：** 已安装 Codex CLI（`npm i -g @openai/codex` 或 `brew install codex`）

</details>

<details>
<summary>autonomous-skill - 长时间任务自动化</summary>

### [autonomous-skill](plugins/autonomous-skill)

使用双代理模式（初始化器 + 执行器）跨多个会话执行复杂、长时间运行的任务，并具有自动会话继续功能。

**安装：**

```sh
/plugin marketplace add feiskyer/claude-code-settings
/plugin install autonomous-skill
```

**主要功能：**

- 双代理模式（初始化器创建任务列表，执行器完成任务）
- 带有进度跟踪的跨会话自动继续
- 使用每个任务目录进行任务隔离（`.autonomous/<任务名称>/`）
- 通过 `task_list.md` 和 `progress.md` 实现进度持久化
- 使用 Claude CLI 的无头模式执行

**使用方法：**

```text
您："请使用 autonomous skill 为待办事项应用构建 REST API"
Claude：[创建 .autonomous/build-rest-api-todo/、初始化任务列表、开始执行]
```

**要求：** 已安装 Claude CLI

</details>

<details>
<summary>nanobanana-skill - 使用 Gemini nanobanana 绘制图像</summary>

### [nanobanana-skill](plugins/nanobanana-skill)

通过 nanobanana 使用 Google Gemini API 生成或编辑图像。在创建、生成或编辑图像时使用。

**安装：**

```sh
/plugin marketplace add feiskyer/claude-code-settings
/plugin install nanobanana-skill
```

**主要功能：**

- 支持各种宽高比的图像生成
- 图像编辑功能
- 多种模型选项（gemini-3-pro-image-preview、gemini-2.5-flash-image）
- 分辨率选项（1K、2K、4K）
- 支持各种宽高比（方形、人像、风景、超宽）

**要求：**

- 在 `~/.nanobanana.env` 中配置 GEMINI_API_KEY
- Python3 与 google-genai、Pillow、python-dotenv（通过在插件目录中运行 `pip install -r requirements.txt` 安装）

</details>

<details>
<summary>youtube-transcribe-skill - 提取 YouTube 字幕</summary>

### [youtube-transcribe-skill](plugins/youtube-transcribe-skill)

从 YouTube 视频链接提取字幕/文本。

**安装：**

```sh
/plugin marketplace add feiskyer/claude-code-settings
/plugin install youtube-transcribe-skill
```

**主要功能：**

- 双重提取方法：CLI（快速）和浏览器自动化（后备）
- 自动字幕语言选择（zh-Hans、zh-Hant、en）
- 浏览器方法的高效基于 DOM 的提取
- 将文本保存到本地文本文件

**要求：**

- `yt-dlp`（用于 CLI 方法）
- 或 `chrome-devtools-mcp`（用于浏览器自动化方法）

</details>

<details>
<summary>kiro-skill - 交互式功能开发</summary>

### [kiro-skill](./skills/kiro-skill)

从想法到实现的交互式功能开发工作流。

**触发方式：** "kiro" 或对 `.kiro/specs/` 目录的引用

**安装：**

```sh
/plugin marketplace add feiskyer/claude-code-settings
/plugin install kiro-skill
```

**工作流**：

1. **需求** → 定义需要构建的内容（EARS 格式的用户故事）
2. **设计** → 确定如何构建（架构、组件、数据模型）
3. **任务** → 创建可执行的实施步骤（测试驱动、增量式）
4. **执行** → 一次实现一个任务

**使用方法**：

```text
您："我需要为用户身份验证创建一个 kiro 功能规格"
Claude：[自动使用 kiro-skill]
```

</details>

<details>
<summary>spec-kit-skill - 基于宪法的开发</summary>

### [spec-kit-skill](./skills/spec-kit-skill)

GitHub Spec-Kit 集成，用于基于宪法的规格驱动开发。

**触发方式：** "spec-kit"、"speckit"、"constitution"、"specify" 或对 `.specify/` 目录的引用

**安装：**

```sh
/plugin marketplace add feiskyer/claude-code-settings
/plugin install spec-kit-skill
```

**先决条件**：

```sh
# 安装 spec-kit CLI
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git

# 初始化项目
specify init . --ai claude
```

**7 阶段工作流**：

1. **宪法** → 建立治理原则
2. **规格** → 定义功能需求
3. **澄清** → 解决歧义（最多 5 个问题）
4. **计划** → 创建技术策略
5. **任务** → 生成依赖排序的任务
6. **分析** → 验证一致性（只读）
7. **实施** → 执行实施

**使用方法**：

```text
您："让我们为此项目创建一个宪法"
Claude：[自动使用 spec-kit-skill，检测 CLI，引导完成各个阶段]
```

</details>

## 代理

`agents/` 目录包含扩展 Claude Code 功能的专用 AI [子代理](https://docs.anthropic.com/en/docs/claude-code/sub-agents)。

<details>
<summary>可用代理</summary>

- **pr-reviewer** - GitHub 拉取请求的专家代码审查员
- **github-issue-fixer** - GitHub 问题解决专家
- **instruction-reflector** - 分析和改进 Claude Code 指令
- **deep-reflector** - 全面的会话分析和学习捕获
- **insight-documenter** - 技术突破文档专家
- **ui-engineer** - UI/UX 开发专家
- **command-creator** - 创建新 Claude Code 自定义命令的专家

</details>

## 设置

[示例设置](settings/README.md) - 各种模型提供商和设置的预配置设置。

<details>
<summary>可用设置</summary>

### [copilot-settings.json](settings/copilot-settings.json)

使用 GitHub Copilot 代理的 Claude Code。指向 localhost:4141 作为 Anthropic API 基础 URL。

### [litellm-settings.json](settings/litellm-settings.json)

使用 LiteLLM 网关的 Claude Code。指向 localhost:4000 作为 Anthropic API 基础 URL。

### [deepseek-settings.json](settings/deepseek-settings.json)

使用 DeepSeek v3.1 的 Claude Code（通过 DeepSeek 的官方 Anthropic 兼容 API）。

### [qwen-settings.json](settings/qwen-settings.json)

使用阿里巴巴 DashScope API 的 Qwen 模型的 Claude Code。通过 claude-code-proxy 使用 Qwen3-Coder-Plus 模型。

### [siliconflow-settings.json](settings/siliconflow-settings.json)

使用 SiliconFlow API 的 Claude Code。使用 Moonshot AI Kimi-K2-Instruct 模型。

### [vertex-settings.json](settings/vertex-settings.json)

使用 Google Cloud Vertex AI 的 Claude Code。使用 Claude Opus 4 模型和 Google Cloud 项目设置。

### [azure-settings.json](settings/azure-settings.json)

使用 Azure AI 的 Claude Code 配置（Anthropic 兼容端点）。指向 Azure AI 服务端点。

### [azure-foundry-settings.json](settings/azure-foundry-settings.json)

使用 Azure AI Foundry 本机模式的 Claude Code 配置。使用 `CLAUDE_CODE_USE_FOUNDRY` 标志和 Claude Opus 4.1 + Sonnet 4.5 模型。

### [minimax.json](settings/minimax.json)

使用 MiniMax API 的 Claude Code 配置。使用 MiniMax-M2 模型。

### [openrouter-settings.json](settings/openrouter-settings.json)

使用 OpenRouter API 的 Claude Code。OpenRouter 通过统一 API 访问许多模型。注意：`ANTHROPIC_API_KEY` 必须为空，而 `ANTHROPIC_AUTH_TOKEN` 包含您的 OpenRouter API 密钥。

</details>

## 限制

Claude Code 中的 **WebSearch** 工具是一个 [Anthropic 专用工具](https://docs.anthropic.com/en/docs/agents-and-tools/tool-use/web-search-tool)，当您不使用官方 Anthropic API 时不可用。因此，如果您需要网络搜索，则需要将 Claude Code 与外部网络搜索 MCP 服务器连接，例如 [Tavily MCP](https://docs.tavily.com/documentation/mcp)、[Brave MCP](https://github.com/brave/brave-search-mcp-server)、[Firecrawl MCP](https://docs.firecrawl.dev/mcp-server) 或 [DuckDuckGo Search MCP](https://github.com/nickclyde/duckduckgo-mcp-server)。

## 常见问题

<details>
<summary>VSCode 中 Claude Code 2.0+ 扩展的登录问题</summary>

对于 VSCode 中的 Claude Code 2.0+ 扩展，如果您不使用 Claude.ai 订阅，请手动将环境变量放在您的 vscode settings.json 中：

```json
{
  "claude-code.environmentVariables": [
    {
      "name": "ANTHROPIC_BASE_URL",
      "value": "http://localhost:4000"
    },
    {
      "name": "ANTHROPIC_AUTH_TOKEN",
      "value": "sk-dummy"
    },
    {
      "name": "ANTHROPIC_MODEL",
      "value": "opusplan"
    },
    {
      "name": "ANTHROPIC_DEFAULT_SONNET_MODEL",
      "value": "claude-sonnet-4.5"
    },
    {
      "name": "ANTHROPIC_DEFAULT_OPUS_MODEL",
      "value": "claude-opus-4"
    },
    {
      "name": "ANTHROPIC_DEFAULT_HAIKU_MODEL",
      "value": "gpt-5-mini"
    },
    {
      "name": "DISABLE_NON_ESSENTIAL_MODEL_CALLS",
      "value": "1"
    },
    {
      "name": "DISABLE_TELEMETRY",
      "value": "1"
    },
    {
      "name": "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC",
      "value": "1"
    }
  ]
}
```

注意，还需要 [~/.claude/config.json](config.json) 的内容来跳过 claude.ai 登录。

</details>

<details>
<summary>缺少 API 密钥和无效 API 密钥问题</summary>

确保您在 `ANTHROPIC_AUTH_TOKEN` 中配置的 API 密钥已添加到 `~/.claude.json` 中的批准 API 密钥，例如

```javascript
{
  "customApiKeyResponses": {
    "approved": [
      "sk-dummy"
    ],
    "rejected": []
  },
  ... （您的其他设置）
}
```

</details>

## 指导

- [使用 GitHub Copilot 作为模型提供商的 Claude Code](guidances/github-copilot.md)。
- [使用 LLM 网关 (LiteLLM) 作为模型提供商的 Claude Code](guidances/llm-gateway-litellm.md)。

## 参考资料

- [Claude Code 官方文档](https://docs.anthropic.com/en/docs/claude-code/overview) - 必读官方文档。
- [anthropics/skills](https://github.com/anthropics/skills) - Claude Code 技能的官方列表，教 Claude 如何以可重复的方式完成特定任务
- [hesreallyhim/awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code) - 策划的斜杠命令、CLAUDE.md 文件、CLI 工具和其他资源列表。
- [wshobson/agents](https://github.com/wshobson/agents) - Claude Code 的全面专用 AI 子代理集合。

## 许可证

本项目在 MIT 许可证下发布 - 详见 [LICENSE](LICENSE)。
