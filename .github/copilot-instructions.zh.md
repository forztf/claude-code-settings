# CLAUDE.md

此文件为 Claude Code 和 GitHub Copilot 在此仓库中工作时提供指导。

## 环境设置

此仓库包含 Claude Code 设置、配置和指导。默认设置使用 GitHub Copilot 作为模型提供商，通过代理 API 访问。

### 必需依赖

- `copilot-api`：使用 `npm install -g copilot-api` 全局安装
- 运行 `copilot-api start --proxy-env` 来授权 GitHub Copilot 账户
- 使用 tmux 进行会话管理：`tmux new-session -d -s copilot 'copilot-api start --proxy-env'`

### 配置

- `settings.json`：包含 API 配置的环境变量
- 使用 `localhost:4141` 作为 API 基础 URL
- 配置使用 `claude-sonnet-4.5` 作为主要模型
- 遥测和非必要流量已禁用

## 自定义命令

Claude Code 的自定义命令按类别组织在 `commands/` 目录中：

### 命令文件结构

所有命令文件必须包含 YAML 前置元数据：

```yaml
---
description: 命令的简要描述
argument-hint: [期望的参数格式]
allowed-tools: 命令可以使用的工具列表
---
```

**必需字段：**

- `description`：对命令用途的清晰简明的描述
- `argument-hint`：显示期望参数的格式（例如 `[问题或疑问]`）
- `allowed-tools`：工具的逗号分隔列表（例如 `Read, Edit, Write, Bash(*)`）

### 命令文档

- 命令文档应放在 `README.md` 的 Commands 部分
- 使用可折叠的 `<details>` 部分，配以清晰的摘要
- 包含使用示例和主要功能
- 切勿在命令文件内部记录命令

### 目录结构

```sh
commands/
├── cc/          # Claude Code 命令
│   └── create-command.md
└── gh/          # GitHub 命令
    └── review-pr.md
```

### 使用方法

使用斜杠语法运行命令：

```sh
/[类别:][命令] [参数]
```

**示例：**

- `/cc:create-command mycommand` - 创建新命令
- `/gh:review-pr 123` - 审查拉取请求 #123

所有可用命令都应在 README.md 的 Commands 部分中记录。

### 提示工程原则

创建或修改命令提示时：

- **系统化构建提示**：使用清晰的阶段或部分（例如分析阶段、执行阶段）
- **定义具体输出**：包含明确的输出格式和结构
- **提供有条理的方法**：将复杂任务分解为编号步骤或项目符号
- **包含元认知元素**：添加偏见意识、假设检查和不确定性评估
- **平衡全面性与简洁性**：在保持清晰度的同时追求全面分析
- **使用渐进式复杂性**：从简单概念开始，逐步构建到更复杂的概念

### 行为准则

- **简洁沟通**：提供直接答案，无需不必要的前言或阐述
- **遵循现有模式**：始终检查类似命令以保持一致的结构和方法
- **优先编辑而非创建**：始终编辑现有文件而非创建新文件，除非绝对必要
- **对复杂任务使用 TodoWrite**：跟踪多步骤流程并确保完成所有要求

## 指导文档

Claude Code 的指导文档应放在 `guidances/` 目录下，并在 README.md 中链接。
