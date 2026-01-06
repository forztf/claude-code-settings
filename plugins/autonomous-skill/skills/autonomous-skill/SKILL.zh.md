---
name: autonomous-skill
description: 当用户想要执行需要多个会话才能完成的长时间运行任务时使用。此技能管理任务分解、进度跟踪和使用 Claude Code 无头模式自动继续的自主执行。触发短语："autonomous"、"long-running task"、"multi-session"、"自主执行"、"长时任务"、"autonomous skill"。
allowed-tools: Read, Write, Edit, Glob, Grep, Task, Bash(cat:*), Bash(ls:*), Bash(tree:*), Bash(mkdir:*), Bash(touch:*), Bash(pwd:*), Bash(cd:*), Bash(grep:*), Bash(find:*), Bash(head:*), Bash(tail:*), Bash(claude:*)
---

# 自主技能 - 长时间任务执行

使用双代理模式（初始化器 + 执行器）跨多个会话执行复杂、长时间运行的任务，并具有自动会话继续功能。

## 目录结构

所有任务数据存储在项目根目录下的 `.autonomous/<任务名称>/` 中：

```
project-root/
└── .autonomous/
    ├── build-rest-api/
    │   ├── task_list.md
    │   └── progress.md
    ├── refactor-auth/
    │   ├── task_list.md
    │   └── progress.md
    └── ...
```

这允许多个自主任务并行运行而不会产生冲突。

## 工作流概述

```
用户请求 → 生成任务名称 → 创建 .autonomous/<任务名称>/ → 执行会话
```

## 步骤 1：初始化任务目录

从用户的描述生成任务名称并创建目录：

```bash
# 生成任务名称（小写、连字符、最多 30 个字符）
# 示例："为待办事项应用构建 REST API" → "build-rest-api-todo"
TASK_NAME=$(echo "$USER_TASK" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9]/-/g' | sed 's/--*/-/g' | cut -c1-30 | sed 's/-$//')

# 创建任务目录
TASK_DIR=".autonomous/$TASK_NAME"
mkdir -p "$TASK_DIR"

echo "任务目录：$TASK_DIR"
```

## 步骤 2：分析当前状态

检查这是新任务还是继续任务：

```bash
TASK_DIR=".autonomous/$TASK_NAME"

# 查找现有任务列表
if [ -f "$TASK_DIR/task_list.md" ]; then
  echo "=== 继续模式 ==="
  echo "在以下位置找到现有任务：$TASK_DIR"

  # 显示进度摘要
  TOTAL=$(grep -c '^\- \[' "$TASK_DIR/task_list.md" 2>/dev/null || echo "0")
  DONE=$(grep -c '^\- \[x\]' "$TASK_DIR/task_list.md" 2>/dev/null || echo "0")
  echo "进度：已完成 $DONE/$TOTAL 个任务"

  # 显示最近的进度说明
  echo ""
  echo "=== 最近进度 ==="
  head -50 "$TASK_DIR/task_list.md"
else
  echo "=== 新任务模式 ==="
  echo "在以下位置创建新任务：$TASK_DIR"
  mkdir -p "$TASK_DIR"
fi
```

## 步骤 3：选择代理模式

### 对于新任务（初始化器模式）

如果任务目录中不存在 `task_list.md`：

```bash
SKILL_DIR="${CLAUDE_PLUGIN_ROOT}/skills/autonomous-skill"
TASK_DIR=".autonomous/$TASK_NAME"

# 读取初始化器提示模板
INITIALIZER_PROMPT=$(cat "$SKILL_DIR/templates/initializer-prompt.md")

# 执行初始化器会话
claude -p "任务：$USER_TASK_DESCRIPTION
任务目录：$TASK_DIR

$INITIALIZER_PROMPT" \
  --output-format stream-json \
  --max-turns 50 \
  --append-system-prompt "您是初始化器代理。在 $TASK_DIR 目录中创建 task_list.md 和 progress.md。"
```

### 对于继续任务（执行器模式）

如果任务目录中存在 `task_list.md`：

```bash
SKILL_DIR="${CLAUDE_PLUGIN_ROOT}/skills/autonomous-skill"
TASK_DIR=".autonomous/$TASK_NAME"

# 读取执行器提示模板
EXECUTOR_PROMPT=$(cat "$SKILL_DIR/templates/executor-prompt.md")

# 读取当前状态
TASK_LIST=$(cat "$TASK_DIR/task_list.md")
PROGRESS=$(cat "$TASK_DIR/progress.md" 2>/dev/null || echo "没有之前的进度说明")

# 执行执行器会话
claude -p "继续处理任务。
任务目录：$TASK_DIR

当前 task_list.md：
$TASK_LIST

之前的进度说明：
$PROGRESS

$EXECUTOR_PROMPT" \
  --output-format stream-json \
  --max-turns 100 \
  --append-system-prompt "您是执行器代理。完成 $TASK_DIR 目录中的任务并更新文件。"
```

## 步骤 4：自动继续循环

每次会话完成后，检查剩余任务并自动继续：

```bash
#!/bin/bash
TASK_DIR=".autonomous/$TASK_NAME"
AUTO_CONTINUE_DELAY=3
SESSION_NUM=1

while true; do
  echo ""
  echo "=========================================="
  echo "  会话 $SESSION_NUM - 任务：$TASK_NAME"
  echo "=========================================="

  # 运行适当的代理
  # ... 执行会话 ...

  # 检查完成情况
  if [ -f "$TASK_DIR/task_list.md" ]; then
    TOTAL=$(grep -c '^\- \[' "$TASK_DIR/task_list.md" 2>/dev/null || echo "0")
    DONE=$(grep -c '^\- \[x\]' "$TASK_DIR/task_list.md" 2>/dev/null || echo "0")

    echo ""
    echo "=== 进度：已完成 $DONE/$TOTAL 个任务 ==="

    if [ "$DONE" -eq "$TOTAL" ] && [ "$TOTAL" -gt 0 ]; then
      echo ""
      echo "所有任务已完成！退出。"
      break
    fi
  fi

  # 延迟后自动继续
  echo ""
  echo "$AUTO_CONTINUE_DELAY 秒后继续...（按 Ctrl+C 暂停）"
  sleep $AUTO_CONTINUE_DELAY

  SESSION_NUM=$((SESSION_NUM + 1))
done
```

## 步骤 5：报告进度

执行后，显示清晰的进度报告：

```
==========================================
  会话完成 - 任务：build-rest-api
==========================================

任务目录：.autonomous/build-rest-api/

本次会话完成的任务：
- [x] 任务 5：实现用户身份验证
- [x] 任务 6：添加登录表单验证

总体进度：18/50 个任务 (36%)

下一步任务：
- [ ] 任务 7：创建密码重置流程
- [ ] 任务 8：添加会话管理

3 秒后继续...（按 Ctrl+C 暂停）
```

## 使用示例

### 示例 1：开始新任务

```
用户：请使用自主技能为待办事项应用构建 REST API

响应：
1. 生成的任务名称："build-rest-api-todo"
2. 创建的目录：.autonomous/build-rest-api-todo/
3. 正在运行初始化器代理...
4. 创建了包含 25 个任务的 task_list.md
5. 进度：已完成 3/25
6. 3 秒后自动继续...
```

### 示例 2：继续现有任务

```
用户：继续自主任务 "build-rest-api-todo"

响应：
1. 找到任务：.autonomous/build-rest-api-todo/
2. 当前进度：15/25 个任务
3. 正在运行执行器代理...
4. 完成任务 16-17
5. 进度：已完成 17/25
6. 3 秒后自动继续...
```

### 示例 3：列出所有任务

```bash
# 列出所有自主任务
ls -la .autonomous/

# 显示特定任务的进度
cat .autonomous/build-rest-api-todo/task_list.md
```

## 关键文件

对于 `.autonomous/<任务名称>/` 中的每个任务：

| 文件 | 用途 |
|------|---------|
| `task_list.md` | 带有复选框进度的主要任务列表 |
| `progress.md` | 逐个会话的进度说明 |

## 重要说明

1. **任务隔离**：每个任务都有自己的目录，不会产生冲突
2. **任务命名**：从描述自动生成（小写、连字符）
3. **任务列表是神圣的**：永远不要删除或修改描述，只标记 `[x]`
4. **每个会话一次一个任务**：专注于彻底完成任务
5. **自动继续**：会话自动继续，延迟 3 秒；按 Ctrl+C 暂停

## 故障排除

| 问题 | 解决方案 |
|-------|----------|
| 未找到任务 | 检查 `.autonomous/` 中的现有任务 |
| 多个任务 | 明确指定任务名称 |
| 会话卡住 | 检查任务目录中的 `progress.md` |
| 需要重新启动 | 删除任务目录并重新开始
