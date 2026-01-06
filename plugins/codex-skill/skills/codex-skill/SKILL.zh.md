---
name: codex-skill
description: 利用 OpenAI Codex/GPT 模型进行自主代码实施。触发："codex"、"use gpt"、"gpt-5"、"gpt-5.2"、"let openai"、"full-auto"、"用codex"、"让gpt实现"。
allowed-tools: Read, Write, Glob, Grep, Task, Bash(cat:*), Bash(ls:*), Bash(tree:*), Bash(codex:*), Bash(codex *), Bash(which:*), Bash(npm:*), Bash(brew:*)
---

# Codex

您正在 **codex exec** 模式下运行 - 一种用于免提任务执行的非交互式自动化模式。

## 先决条件

在使用此技能之前，确保已安装并配置 Codex CLI：

1. **安装验证**：

   ```bash
   codex --version
   ```

2. **首次设置**：如果未安装，引导用户使用命令 `npm i -g @openai/codex` 或 `brew install codex` 安装 Codex CLI。

## 核心原则

### 自主执行

- 从头到尾执行任务，无需为每个操作寻求批准
- 根据最佳实践和任务要求做出自信的决策
- 仅在真正缺少关键信息时才提问
- 优先完成工作流程，而不是解释每个步骤

### 输出行为

- 在工作时流式传输进度更新
- 完成后提供清晰、结构化的最终摘要
- 专注于可操作的结果和指标，而不是冗长的解释
- 报告已做的事情，而不是可能做的事情

### 操作模式

Codex 使用沙箱策略来控制允许的操作：

**只读模式（默认）**

- 分析代码、搜索文件、读取文档
- 提供见解、建议和执行计划
- 不修改代码库
- 适合探索和分析任务
- **这是运行 `codex exec` 时的默认模式**

**工作区写入模式（推荐用于编程）**

- 在工作区内读取和写入文件
- 实施功能、修复错误、重构代码
- 在工作区内创建、修改和删除文件
- 执行构建命令和测试
- **使用 `--full-auto` 或 `-s workspace-write` 启用文件编辑**
- **这是大多数编程任务的推荐模式**

**危险完全访问模式**

- 所有工作区写入功能
- 用于获取依赖的网络访问
- 工作区外的系统级操作
- 对系统上所有文件的访问
- **仅在明确请求和必要时使用**
- 使用标志：`-s danger-full-access` 或 `--sandbox danger-full-access`

## Codex CLI 命令

**注意**：以下命令包括 Codex exec 文档中记录的功能和 CLI 中可用的其他标志（通过 `codex exec --help` 验证）。

### 模型选择

当用户要求时，使用 `-m` 或 `--model` 指定要使用的模型（未指定 -m/--model 时使用默认模型）：

```bash
codex exec -m gpt-5.2 "重构支付处理模块"
codex exec -m gpt-5.2-codex "实施用户身份验证功能"
codex exec -m gpt-5.2-codex-max "分析代码库架构"
```

### 沙箱模式

使用 `-s` 或 `--sandbox` 控制执行权限（可能的值：read-only、workspace-write、danger-full-access）：

#### 只读模式

```bash
codex exec -s read-only "分析代码库结构并统计代码行数"
codex exec --sandbox read-only "审查代码质量并提出改进建议"
```

在不进行任何修改的情况下分析代码。

#### 工作区写入模式（推荐用于编程）

```bash
codex exec -s workspace-write "实施用户身份验证功能"
codex exec --sandbox workspace-write "修复登录流程中的错误"
```

在工作区内读取和写入文件。**必须显式启用（不是默认值）。** 将此用于大多数编程任务。

#### 危险完全访问模式

```bash
codex exec -s danger-full-access "安装依赖项并更新 API 集成"
codex exec --sandbox danger-full-access "使用 npm 包设置开发环境"
```

网络访问和系统级操作。仅在必要时使用。

### 全自动模式（便捷别名）

```bash
codex exec --full-auto "实施用户身份验证功能"
```

**便捷别名**：`-s workspace-write`（启用文件编辑）。
这是**大多数编程任务的推荐命令**，因为它允许 codex 更改您的代码库。

### 配置配置文件

使用 `~/.codex/config.toml` 中保存的配置文件，使用 `-p` 或 `--profile`（如果您的版本支持）：

```bash
codex exec -p production "部署最新更改"
codex exec --profile development "运行集成测试"
```

配置文件可以指定默认模型、沙箱模式和其他选项。
*通过 `codex exec --help` 验证可用性*

### 工作目录

使用 `-C` 或 `--cd` 指定不同的工作目录（如果您的版本支持）：

```bash
codex exec -C /path/to/project "实施功能"
codex exec --cd ~/projects/myapp "运行测试并修复失败"
```

*通过 `codex exec --help` 验证可用性*

### 附加可写目录

使用 `--add-dir` 允许在主工作区之外的其他目录中进行写入（如果您的版本支持）：

```bash
codex exec --add-dir /tmp/output --add-dir ~/shared "在多个位置生成报告"
```

当任务需要写入特定的外部目录时，这很有用。
*通过 `codex exec --help` 验证可用性*

### JSON 输出

```bash
codex exec --json "运行测试并报告结果"
codex exec --json -s read-only "分析安全漏洞"
```

输出具有推理、命令、文件更改和指标的结构化 JSON Lines 格式。

### 将输出保存到文件

```bash
codex exec -o report.txt "生成安全审计报告"
codex exec -o results.json --json "运行性能基准测试"
```

将最终消息写入文件而不是 stdout。

### 跳过 Git 仓库检查

```bash
codex exec --skip-git-repo-check "分析这个非 git 目录"
```

绕过目录必须是 git 仓库的要求。

### 恢复上一个会话

```bash
codex exec resume --last "现在实施下一个功能"
```

恢复上一个会话并继续执行新任务。

### 绕过批准和沙箱（如果可用）

**⚠️ 警告：使用前请验证此标志是否存在 ⚠️**

某些版本的 Codex 可能支持 `--dangerously-bypass-approvals-and-sandbox`：

```bash
codex exec --dangerously-bypass-approvals-and-sandbox "执行任务"
```

**如果此标志可用**：
- 跳过所有确认提示
- 在无沙箱的情况下执行命令
- 应仅在外部沙箱环境（容器、虚拟机）中使用
- **极其危险 - 永远不要在您的开发机器上使用**

**首先验证可用性**：运行 `codex exec --help` 以检查您的版本是否支持此标志。

### 组合示例

为复杂场景组合多个标志：

```bash
# 使用特定模型和工作区写入以及 JSON 输出
codex exec -m gpt-5.1-codex -s workspace-write --json "实施身份验证并输出结果"

# 使用配置文件和自定义工作目录
codex exec -p production -C /var/www/app "部署更新"

# 全自动模式和附加目录以及输出文件
codex exec --full-auto --add-dir /tmp/logs -o summary.txt "重构并记录更改"

# 使用特定模型在不同目录中跳过 git 检查
codex exec -m gpt-5.1-codex -C ~/non-git-project --skip-git-repo-check "分析并改进代码"
```

## 执行工作流

1. **解析请求**：了解完整的目标和范围
2. **高效规划**：创建最小、专注的执行计划
3. **自主执行**：自信地实施解决方案
4. **验证结果**：根据需要运行测试、检查或验证
5. **清晰报告**：提供已完成工作的结构化摘要

## 最佳实践

### 速度和效率

- 当次要细节模棱两可时做出合理的假设
- 尽可能使用并行操作（读取多个文件、运行多个命令）
- 避免冗长的解释 - 专注于做
- 不要为标准操作寻求确认

### 范围管理

- 严格专注于请求的任务
- 不要添加未请求的功能或改进
- 避免重构不属于任务的一部分代码
- 保持解决方案最小化和直接

### 质量标准

- 遵循现有的代码模式和约定
- 进行更改后运行相关测试
- 验证解决方案确实有效
- 报告遇到的任何错误或限制

## 何时中断执行

仅在遇到以下情况时暂停以获取用户输入：

- **破坏性操作**：删除数据库、强制推送到主分支、删除表
- **安全决策**：暴露凭据、更改身份验证、打开端口
- **模棱两可的要求**：多种有效方法且具有重大权衡
- **缺少关键信息**：无法在没有用户特定数据的情况下继续

对于所有其他决策，使用最佳判断自主进行。

## 最终输出格式

始终以结构化摘要结束：

```
✓ 任务成功完成

所做的更改：
- [修改/创建的文件列表]
- [关键代码更改]

结果：
- [指标：更改的行数、受影响的文件、运行的测试]
- [以前不起作用但现在起作用的东西]

验证：
- [运行的测试、执行的检查]

后续步骤（如适用）：
- [后续任务的建议]
```

## 示例使用场景

### 代码分析（只读）

**用户**："按语言统计此项目中的代码行数"
**模式**：只读
**命令**：

```bash
codex exec -s read-only "按语言统计此项目中的代码总行数"
```

**操作**：搜索所有文件，按扩展名分类，统计行数，报告总计

### 错误修复（工作区写入）

**用户**："使用 gpt-5 修复登录流程中的身份验证错误"
**模式**：工作区写入
**命令**：

```bash
codex exec -m gpt-5 --full-auto "修复登录流程中的身份验证错误"
```

**操作**：查找错误，实施修复，运行测试，提交更改

### 功能实施（工作区写入）

**用户**："让 codex 为 UI 实施暗模式支持"
**模式**：工作区写入
**命令**：

```bash
codex exec --full-auto "使用主题上下文和样式更新为 UI 添加暗模式支持"
```

**操作**：识别组件，添加主题上下文，更新样式，在两种模式下测试

### 批量操作（工作区写入）

**用户**："让 gpt-5.1 将所有从 old-lib 到 new-lib 的导入更新"
**模式**：工作区写入
**命令**：

```bash
codex exec -m gpt-5.1 -s workspace-write "在整个代码库中将所有从 old-lib 到 new-lib 的导入更新"
```

**操作**：查找所有导入，执行替换，验证语法，运行测试

### 生成 JSON 输出报告（只读）

**用户**："分析安全漏洞并以 JSON 格式输出"
**模式**：只读
**命令**：

```bash
codex exec -s read-only --json "分析代码库的安全漏洞并提供详细报告"
```

**操作**：扫描代码，识别问题，输出包含发现的结构化 JSON

### 安装依赖项并集成 API（危险完全访问）

**用户**："安装新的支付 SDK 并集成它"
**模式**：危险完全访问
**命令**：

```bash
codex exec -s danger-full-access "安装支付 SDK 依赖项并集成 API"
```

**操作**：安装包，更新代码，添加集成点，测试功能

### 多项目工作（自定义目录）

**用户**："使用 codex 在后端项目中实施 API"
**模式**：工作区写入
**命令**：

```bash
codex exec -C ~/projects/backend --full-auto "实施用户管理的 REST API 端点"
```

**操作**：切换到后端目录，实施 API 端点，编写测试

### 使用日志进行重构（附加目录）

**用户**："重构数据库层并记录更改"
**模式**：工作区写入
**命令**：

```bash
codex exec --full-auto --add-dir /tmp/refactor-logs "为了更好的性能重构数据库层并记录所有更改"
```

**操作**：重构代码，将日志写入外部目录，运行测试

### 生产部署（使用配置文件）

**用户**："使用生产配置文件进行部署"
**模式**：基于配置文件
**命令**：

```bash
codex exec -p production "将最新更改部署到生产环境"
```

**操作**：使用生产配置，部署代码，验证部署

### 非 Git 项目分析

**用户**："分析这个不在 git 中的遗留代码库"
**模式**：只读
**命令**：

```bash
codex exec -s read-only --skip-git-repo-check "分析架构并提出现代化方法"
```

**操作**：分析代码结构，提供现代化建议

## 错误处理

发生错误时：

1. 尝试自动恢复（如果可能）
2. 在输出中清晰地记录错误
3. 如果错误是非阻塞的，继续执行剩余任务
4. 在最终摘要中报告所有错误
5. 仅在错误使继续不可能时停止

## 可恢复执行

如果执行被中断：

- 清楚地说明已完成的内容
- 提供恢复的确切命令/步骤
- 列出任何需要保留的状态
- 解释剩余需要做的事情
