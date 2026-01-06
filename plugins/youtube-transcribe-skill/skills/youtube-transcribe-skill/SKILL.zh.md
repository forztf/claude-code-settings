---
name: youtube-transcribe-skill
description: 从 YouTube 视频中提取字幕/转录。触发："youtube transcript"、"extract subtitles"、"video captions"、"视频字幕"、"字幕提取"、"YouTube转文字"、"提取字幕"。
allowed-tools: Read, Write, Glob, Grep, Task, Bash(cat:*), Bash(ls:*), Bash(tree:*), Bash(yt-dlp:*), Bash(which:*), mcp__plugin_claude-code-settings_chrome__*
---

# YouTube 字幕提取

从 YouTube 视频 URL 提取字幕/转录并将其保存为本地文件。

输入 YouTube URL：$ARGUMENTS

## 步骤 1：验证 URL 并获取视频信息

1. **验证 URL 格式**：确认输入是有效的 YouTube URL（支持 `youtube.com/watch?v=` 或 `youtu.be/` 格式）。

2. **获取视频信息**：使用 WebFetch 或 firecrawl 获取页面并提取视频标题以用于后续文件命名。

## 步骤 2：CLI 快速提取（优先尝试）

使用命令行工具快速提取字幕。

1. **检查工具可用性**：
   执行 `which yt-dlp`。

   - 如果**找到** `yt-dlp`，继续进行字幕下载。
   - 如果**未找到** `yt-dlp`，立即跳转到**步骤 3**。

2. **执行字幕下载**（仅当找到 `yt-dlp` 时）：

   - **提示**：始终添加 `--cookies-from-browser` 以避免登录限制。默认为 `chrome`。
   - **重试逻辑**：如果 `yt-dlp` 因浏览器错误失败（例如，"Could not open Chrome"），请询问用户指定其可用的浏览器（例如，`firefox`、`safari`、`edge`）并重试。

   ```bash
   # 首先获取标题（首先尝试 chrome）
   yt-dlp --cookies-from-browser=chrome --get-title "[VIDEO_URL]"

   # 下载字幕
   yt-dlp --cookies-from-browser=chrome --write-auto-sub --write-sub --sub-lang zh-Hans,zh-Hant,en --skip-download --output "<Video Title>.%(ext)s" "[VIDEO_URL]"
   ```

3. **验证结果**：
   - 检查命令退出代码。
   - **退出代码 0（成功）**：字幕已本地保存，任务完成。
   - **退出代码非 0（失败）**：
     - 如果错误与浏览器/cookies 相关，请询问用户正确的浏览器并重试步骤 2。
     - 如果其他错误（例如，视频不可用），继续进行**步骤 3**。

## 步骤 3：浏览器自动化（后备）

当 CLI 方法失败或缺少 `yt-dlp` 时，使用浏览器 UI 自动化提取字幕。

1. **检查工具可用性**：

   - 检查 `chrome-devtools-mcp` 工具（特别是 `mcp__plugin_claude-code-settings_chrome__new_page`）是否可用。
   - **关键检查**：如果 `chrome-devtools-mcp` **不可用**且在步骤 2 中**未找到** `yt-dlp`：
     - **停止**执行。
     - **通知用户**："无法继续。请安装 `yt-dlp`（用于快速 CLI 提取）或配置 `chrome-devtools-mcp`（用于浏览器自动化）。"

2. **初始化浏览器会话**（如果工具可用）：

   调用 `mcp__plugin_claude-code-settings_chrome__new_page` 打开视频 URL。

### 3.2 分析页面状态

调用 `mcp__plugin_claude-code-settings_chrome__take_snapshot` 读取页面可访问性树。

### 3.3 展开视频描述

_原因："显示转录稿"按钮通常隐藏在折叠的描述区域内。_

1. 在快照中搜索标记为 **"...more"**、**"...更多"** 或 **"Show more"** 的按钮（通常位于视频标题下方的描述块中）。
2. 调用 `mcp__plugin_claude-code-settings_chrome__click` 单击该按钮。

### 3.4 打开转录稿面板

1. 调用 `mcp__plugin_claude-code-settings_chrome__take_snapshot` 获取更新的 UI 快照。
2. 搜索标记为 **"Show transcript"**、**"显示转录稿"** 或 **"内容转文字"** 的按钮。
3. 调用 `mcp__plugin_claude-code-settings_chrome__click` 单击该按钮。

### 3.5 通过 DOM 提取内容

_原因：直接读取可访问性树以获取长列表很慢并消耗大量令牌；DOM 注入更高效。_

调用 `mcp__plugin_claude-code-settings_chrome__evaluate_script` 执行以下 JavaScript：

```javascript
() => {
  // 选择所有转录片段容器
  const segments = document.querySelectorAll("ytd-transcript-segment-renderer");
  if (!segments.length) return "BUFFERING"; // 如果为空则重试

  // 迭代并格式化为"timestamp text"
  return Array.from(segments)
    .map((seg) => {
      const time = seg.querySelector(".segment-timestamp")?.innerText.trim();
      const text = seg.querySelector(".segment-text")?.innerText.trim();
      return `${time} ${text}`;
    })
    .join("\n");
};
```

如果返回 "BUFFERING"，请等待几秒钟并重试。

### 3.6 保存和清理

1. 使用 Write 工具将提取的文本保存为本地文件（例如，`<Video Title>.txt`）。
2. 调用 `mcp__plugin_claude-code-settings_chrome__close_page` 释放资源。

## 输出要求

- 将字幕文件保存到当前工作目录。
- 文件名格式：`<Video Title>.txt`
- 文件内容格式：每行应为 `Timestamp Subtitle Text`。
- 完成后报告：文件路径、字幕语言、总行数。
