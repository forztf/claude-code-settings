---
name: nanobanana-skill
description: 通过 nanobanana 使用 Google Gemini API 生成或编辑图像。触发："nanobanana"、"generate image"、"create image"、"edit image"、"AI drawing"、"图片生成"、"AI绘图"、"图片编辑"、"生成图片"。
allowed-tools: Read, Write, Glob, Grep, Task, Bash(cat:*), Bash(ls:*), Bash(tree:*), Bash(python3:*))
---

# Nanobanana 图像生成技能

通过 nanobanana 工具使用 Google Gemini API 生成或编辑图像。

## 要求

1. **GEMINI_API_KEY**：必须在 `~/.nanobanana.env` 中配置或 `export GEMINI_API_KEY=<your-api-key>`
2. **安装了依赖包的 Python3**：google-genai、Pillow、python-dotenv。可以通过 `python3 -m pip install -r ${CLAUDE_PLUGIN_ROOT}/skills/nanobanana-skill/requirements.txt` 安装（如果尚未安装）。
3. **可执行文件**：`${CLAUDE_PLUGIN_ROOT}/skills/nanobanana-skill/nanobanana.py`

## 说明

### 对于图像生成

1. 询问用户：
   - 他们想创建什么（提示词）
   - 期望的宽高比/尺寸（可选，默认为 9:16 人像）
   - 输出文件名（可选，如果未指定则自动生成 UUID）
   - 模型偏好（可选，默认为 gemini-3-pro-image-preview）
   - 分辨率（可选，默认为 1K）

2. 使用适当的参数运行 nanobanana 脚本：

   ```bash
   python3 ${CLAUDE_PLUGIN_ROOT}/skills/nanobanana-skill/nanobanana.py --prompt "图像描述" --output "filename.png"
   ```

3. 完成时向用户显示保存的图像路径

### 对于图像编辑

1. 询问用户：
   - 要编辑的输入图像文件
   - 他们想要什么更改（提示词）
   - 输出文件名（可选）

2. 使用输入图像运行：

   ```bash
   python3 ${CLAUDE_PLUGIN_ROOT}/skills/nanobanana-skill/nanobanana.py --prompt "编辑说明" --input image1.png image2.png --output "edited.png"
   ```

## 可用选项

### 宽高比 (--size)

- `1024x1024` (1:1) - 方形
- `832x1248` (2:3) - 人像
- `1248x832` (3:2) - 风景
- `864x1184` (3:4) - 人像
- `1184x864` (4:3) - 风景
- `896x1152` (4:5) - 人像
- `1152x896` (5:4) - 风景
- `768x1344` (9:16) - 人像（默认）
- `1344x768` (16:9) - 风景
- `1536x672` (21:9) - 超宽

### 模型 (--model)

- `gemini-3-pro-image-preview`（默认）- 更高质量
- `gemini-2.5-flash-image` - 更快生成

### 分辨率 (--resolution)

- `1K`（默认）
- `2K`
- `4K`

## 示例

### 生成简单图像

```bash
python3 ${CLAUDE_PLUGIN_ROOT}/skills/nanobanana-skill/nanobanana.py --prompt "宁静的山湖日落景观"
```

### 使用特定尺寸和输出生成

```bash
python3 ${CLAUDE_PLUGIN_ROOT}/skills/nanobanana-skill/nanobanana.py \
  --prompt "科技初创公司的现代极简主义标志" \
  --size 1024x1024 \
  --output "logo.png"
```

### 生成高分辨率风景图像

```bash
python3 ${CLAUDE_PLUGIN_ROOT}/skills/nanobanana-skill/nanobanana.py \
  --prompt "未来派飞行汽车城市景观" \
  --size 1344x768 \
  --resolution 2K \
  --output "cityscape.png"
```

### 编辑现有图像

```bash
python3 ${CLAUDE_PLUGIN_ROOT}/skills/nanobanana-skill/nanobanana.py \
  --prompt "在天空中添加彩虹" \
  --input photo.png \
  --output "photo-with-rainbow.png"
```

### 使用更快的模型

```bash
python3 ${CLAUDE_PLUGIN_ROOT}/skills/nanobanana-skill/nanobanana.py \
  --prompt "猫的快速草图" \
  --model gemini-2.5-flash-image \
  --output "cat-sketch.png"
```

## 错误处理

如果脚本失败：

- 检查 `GEMINI_API_KEY` 是否已导出或在 ~/.nanobanana.env 中设置
- 验证输入图像文件存在且可读
- 确保输出目录可写
- 如果没有生成图像，请尝试使提示词更具体地说明需要图像

## 最佳实践

1. 在提示词中要描述性 - 包括风格、情绪、颜色、构图
2. 对于标志/图形，使用方形宽高比（1024x1024）
3. 对于社交媒体帖子，故事使用 9:16，帖子使用 1:1
4. 对于壁纸，使用 16:9 或 21:9
5. 测试时从 1K 分辨率开始，最终输出升级到 2K/4K
6. 使用 gemini-3-pro-image-preview 获得最佳质量，使用 gemini-2.5-flash-image 提高速度
