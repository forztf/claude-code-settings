# Spec-Kit 检测逻辑

用于确定项目状态和引导用户完成 spec-kit 工作流的综合检测算法。

## 1. CLI 安装检测

检查系统上是否安装了 `specify` CLI。

### 方法 1：命令检查

```bash
if command -v specify &> /dev/null; then
  echo "通过 PATH 安装 CLI"
  specify --version
  exit 0
fi
```

### 方法 2：直接路径检查

```bash
if [ -x "$HOME/.local/bin/specify" ]; then
  echo "在 ~/.local/bin 中安装 CLI"
  "$HOME/.local/bin/specify" --version
  exit 0
fi
```

### 方法 3：UV 工具检查

```bash
if command -v uv &> /dev/null; then
  if uv tool list | grep -q "specify-cli"; then
    echo "通过 uv tool 安装 CLI"
    uv tool run specify --version
    exit 0
  fi
fi
```

### 组合检测函数

```bash
detect_cli() {
  # 检查命令
  if command -v specify &> /dev/null; then
    echo "installed"
    return 0
  fi

  # 检查本地 bin
  if [ -x "$HOME/.local/bin/specify" ]; then
    echo "installed"
    return 0
  fi

  # 检查 uv tool
  if command -v uv &> /dev/null && uv tool list 2>/dev/null | grep -q "specify-cli"; then
    echo "installed"
    return 0
  fi

  echo "not_installed"
  return 1
}
```

### 安装指导

如果未检测到 CLI，引导用户：

```markdown
未安装 spec-kit CLI。要安装：

**持久安装（推荐）：**
```bash
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git
```

**一次性使用：**
```bash
uvx --from git+https://github.com/github/spec-kit.git specify init .
```

**要求：**
- Python 3.11+
- Git
- uv 包管理器（从 https://docs.astral.sh/uv/ 安装）
```

## 2. 项目初始化检测

检查当前项目是否使用 spec-kit 初始化。

### 主要指标

```bash
check_initialization() {
  # 必须有 .specify 目录
  if [ ! -d ".specify" ]; then
    echo "not_initialized"
    return 1
  fi

  # 必须有宪法
  if [ ! -f ".specify/memory/constitution.md" ]; then
    echo "partially_initialized"
    return 2
  fi

  # 必须有脚本
  if [ ! -d ".specify/scripts/bash" ]; then
    echo "partially_initialized"
    return 2
  fi

  # 必须有模板
  if [ ! -d ".specify/templates" ]; then
    echo "partially_initialized"
    return 2
  fi

  echo "initialized"
  return 0
}
```

### 初始化指导

如果未初始化：

```bash
# 在当前目录中初始化
specify init . --ai claude

# 初始化新项目
specify init <project-name> --ai claude

# 选项：
# --force：覆盖非空目录
# --no-git：跳过 Git 初始化
# --script ps：生成 PowerShell 脚本（Windows）
```

## 3. 功能检测

识别现有功能和最新功能。

### 列出所有功能

```bash
list_features() {
  if [ ! -d ".specify/specs" ]; then
    echo "未找到功能"
    return 1
  fi

  # 列出编号的功能目录
  ls -d .specify/specs/[0-9]* 2>/dev/null | sort -V
}
```

### 获取最新功能

```bash
get_latest_feature() {
  LATEST=$(ls -d .specify/specs/[0-9]* 2>/dev/null | sort -V | tail -1)

  if [ -z "$LATEST" ]; then
    echo "未找到功能"
    return 1
  fi

  echo "$LATEST"
  return 0
}
```

### 提取功能名称

```bash
get_feature_name() {
  FEATURE_DIR="$1"

  # 从目录名中提取（例如，001-feature-name -> feature-name）
  basename "$FEATURE_DIR" | sed 's/^[0-9]\{3\}-//'
}
```

### 提取功能编号

```bash
get_feature_number() {
  FEATURE_DIR="$1"

  # 提取编号（例如，001-feature-name -> 001）
  basename "$FEATURE_DIR" | grep -o '^[0-9]\{3\}'
}
```

## 4. 阶段检测

确定功能的当前开发阶段。

### 综合阶段检测

```bash
detect_phase() {
  FEATURE_DIR="$1"

  # 阶段 1：宪法
  if [ ! -f ".specify/memory/constitution.md" ]; then
    echo "constitution"
    return 0
  fi

  # 阶段 2：规格
  if [ ! -d "$FEATURE_DIR" ] || [ ! -f "$FEATURE_DIR/spec.md" ]; then
    echo "specify"
    return 0
  fi

  # 阶段 3：澄清
  # 检查规格是否有澄清部分
  if ! grep -q "## Clarifications" "$FEATURE_DIR/spec.md" 2>/dev/null; then
    echo "clarify"
    return 0
  fi

  # 阶段 4：计划
  if [ ! -f "$FEATURE_DIR/plan.md" ]; then
    echo "plan"
    return 0
  fi

  # 阶段 5：任务
  if [ ! -f "$FEATURE_DIR/tasks.md" ]; then
    echo "tasks"
    return 0
  fi

  # 阶段 6/7：分析或实施
  # 检查是否有未完成的任务
  if grep -q "\\- \\[ \\]" "$FEATURE_DIR/tasks.md" 2>/dev/null; then
    echo "implement"
    return 0
  fi

  # 所有任务完成
  echo "complete"
  return 0
}
```

### 特定阶段检查

#### 检查宪法是否存在

```bash
has_constitution() {
  [ -f ".specify/memory/constitution.md" ]
}
```

#### 检查规格是否存在

```bash
has_specification() {
  FEATURE_DIR="$1"
  [ -f "$FEATURE_DIR/spec.md" ]
}
```

#### 检查澄清是否存在

```bash
has_clarifications() {
  FEATURE_DIR="$1"
  grep -q "## Clarifications" "$FEATURE_DIR/spec.md" 2>/dev/null
}
```

#### 检查计划是否存在

```bash
has_plan() {
  FEATURE_DIR="$1"
  [ -f "$FEATURE_DIR/plan.md" ]
}
```

#### 检查任务是否存在

```bash
has_tasks() {
  FEATURE_DIR="$1"
  [ -f "$FEATURE_DIR/tasks.md" ]
}
```

#### 获取未完成任务

```bash
get_incomplete_tasks() {
  FEATURE_DIR="$1"

  if [ ! -f "$FEATURE_DIR/tasks.md" ]; then
    return 1
  fi

  # 查找未完成的任务（- [ ]）
  grep -n "\\- \\[ \\]" "$FEATURE_DIR/tasks.md"
}
```

#### 获取已完成任务计数

```bash
count_completed_tasks() {
  FEATURE_DIR="$1"

  if [ ! -f "$FEATURE_DIR/tasks.md" ]; then
    echo "0"
    return
  fi

  # 统计已完成的任务（- [x]）
  grep -c "\\- \\[x\\]" "$FEATURE_DIR/tasks.md" 2>/dev/null || echo "0"
}
```

#### 获取总任务计数

```bash
count_total_tasks() {
  FEATURE_DIR="$1"

  if [ ! -f "$FEATURE_DIR/tasks.md" ]; then
    echo "0"
    return
  fi

  # 统计所有任务（- [ ] 或 - [x]）
  grep -c "\\- \\[" "$FEATURE_DIR/tasks.md" 2>/dev/null || echo "0"
}
```

## 5. 完整状态报告

生成综合状态报告：

```bash
generate_status_report() {
  echo "=== Spec-Kit 状态报告 ==="
  echo

  # CLI 状态
  echo "CLI 安装："
  CLI_STATUS=$(detect_cli)
  echo "  状态：$CLI_STATUS"
  if [ "$CLI_STATUS" = "installed" ]; then
    specify --version 2>/dev/null | sed 's/^/  /'
  fi
  echo

  # 项目状态
  echo "项目初始化："
  INIT_STATUS=$(check_initialization)
  echo "  状态：$INIT_STATUS"
  echo

  # 宪法状态
  if has_constitution; then
    echo "宪法：✓ 存在"
  else
    echo "宪法：✗ 缺失（运行阶段 1）"
  fi
  echo

  # 功能
  echo "功能："
  FEATURES=$(list_features)
  if [ -z "$FEATURES" ]; then
    echo "  未找到功能"
  else
    echo "$FEATURES" | while read -r FEATURE; do
      FEATURE_NAME=$(get_feature_name "$FEATURE")
      FEATURE_NUM=$(get_feature_number "$FEATURE")
      PHASE=$(detect_phase "$FEATURE")

      echo "  [$FEATURE_NUM] $FEATURE_NAME"
      echo "      阶段：$PHASE"

      if [ "$PHASE" = "implement" ] || [ "$PHASE" = "complete" ]; then
        COMPLETED=$(count_completed_tasks "$FEATURE")
        TOTAL=$(count_total_tasks "$FEATURE")
        echo "      任务：已完成 $COMPLETED/$TOTAL"
      fi
    done
  fi
  echo

  # 当前阶段指导
  LATEST=$(get_latest_feature)
  if [ -n "$LATEST" ]; then
    CURRENT_PHASE=$(detect_phase "$LATEST")
    echo "下一步操作：阶段 $CURRENT_PHASE"
  elif has_constitution; then
    echo "下一步操作：创建第一个功能（阶段 2：specify）"
  else
    echo "下一步操作：创建宪法（阶段 1）"
  fi
}
```

## 使用示例

### 检查并引导用户

```bash
# 检测状态并提供指导
CLI_STATUS=$(detect_cli)

if [ "$CLI_STATUS" = "not_installed" ]; then
  echo "请先安装 spec-kit CLI："
  echo "uv tool install specify-cli --from git+https://github.com/github/spec-kit.git"
  exit 1
fi

INIT_STATUS=$(check_initialization)

if [ "$INIT_STATUS" != "initialized" ]; then
  echo "请初始化项目："
  echo "specify init . --ai claude"
  exit 1
fi

# 生成状态报告
generate_status_report
```

### 确定下一步操作

```bash
# 自动确定下一步做什么
LATEST=$(get_latest_feature)

if [ -z "$LATEST" ]; then
  if has_constitution; then
    echo "准备好创建第一个功能"
    echo "运行：.specify/scripts/bash/create-new-feature.sh --json 'feature-name'"
  else
    echo "需要先创建宪法"
  fi
else
  PHASE=$(detect_phase "$LATEST")
  echo "当前阶段：$PHASE"
  echo "继续功能 $(get_feature_name "$LATEST") 的阶段 $PHASE"
fi
```
