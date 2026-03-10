---
name: xhs-web
description: 操作小红书(Xiaohongshu)网页。当用户要求下载小红书视频/图片、发评论、关注用户、或其他小红书网页操作时触发。
allowed-tools: Bash(agent-browser *), Bash(curl *)
---

# 小红书网页操作（agent-browser）

前提：浏览器已登录小红书账号。可通过以下命令加载已保存的登录状态：

```bash
agent-browser close          # 必须先关闭正在运行的浏览器
agent-browser state load xiaohongshu-auth.json
```

> **注意**：`state load` 要求浏览器未在运行，否则报错。必须先 `agent-browser close` 再加载。

## 打开帖子的正确方式

小红书帖子 URL 需要 `xsec_token` 参数，否则会 404 或被安全拦截。

- **有完整链接（含 xsec_token）**：直接 `agent-browser open "URL"` 即可
- **从用户主页进入帖子**：点击帖子的 `a.cover` 链接（而非第一个隐藏的 `<a>`），它自带 xsec_token
  ```js
  // 获取第 N 个帖子的带 token 链接（data-index 从 0 开始）
  document.querySelector('section[data-index="0"] a.cover').href
  // 点击进入（会触发页面导航到 /explore/笔记ID?xsec_token=...）
  document.querySelector('section[data-index="0"] a.cover').click()
  ```
  > **前提条件**：`section[data-index] a.cover` 选择器**仅在用户主页（/user/profile/...）上有效**，且必须先 `wait 3000` 等页面加载完成。在搜索结果页、explore 页等其他页面上此选择器无效。如果返回 null，先确认：(1) 当前 URL 是否是用户主页 (2) 是否已等待页面加载 (3) 该用户是否有帖子
- **只有小红书号或用户名**：需先搜索 `https://www.xiaohongshu.com/search_result?keyword=小红书号&type=user` 找到用户，再从主页进入帖子
  - 小红书号可以是纯数字，也可以包含字母（如 `sonw2000315`）
  - **注意**：搜索结果页的第一个 `a[href*="/user/profile/"]` 可能是页面侧边栏中已登录账号的"我"入口，而非搜索结果。选择用户时必须跳过侧边栏链接，从搜索结果列表中选取文本包含目标小红书号的用户链接
  ```js
  // 正确做法：从搜索结果中找包含目标小红书号的用户链接
  var allUserLinks = document.querySelectorAll('a[href*="/user/profile/"]');
  var target = null;
  allUserLinks.forEach(function(a) {
    if (a.textContent.includes('目标小红书号') && !target) target = a;
  });
  target.href; // 这才是正确的用户主页链接
  ```

## "下载帖子内容"的含义

当用户要求"下载帖子中的内容/多媒体"时，**仅下载作者发布的主要媒体**：

| 要下载 | 不要下载 |
|--------|----------|
| 帖子轮播区的图片 | 用户头像、评论区表情包 |
| 帖子嵌入的视频 | 按钮图标、UI 元素 |
| | 文字内容、评论文本 |

**提取方法**（图片和视频统一从运行时 JS 对象获取）：
- **图片**：`window.__INITIAL_STATE__.note.noteDetailMap[noteId].note.imageList`，每个元素的 `urlDefault` 即完整图片 URL。**不要从 DOM 提取**（swiper 懒加载会遗漏未翻到的图片）
- **视频**：从 `__INITIAL_STATE__` 的 `<script>` 标签文本中提取 `masterUrl`，选 `_114.mp4` 或 `_115.mp4` 结尾的无水印流

详见 [references/download-images.md](references/download-images.md) 和 [references/download-video.md](references/download-video.md)

## 功能参考

| 功能 | 参考文档 |
|------|----------|
| 发表评论 | [references/comment.md](references/comment.md) |
| 回复指定评论 | [references/reply.md](references/reply.md) |
| 下载帖子图片 | [references/download-images.md](references/download-images.md) |
| 下载无水印视频 | [references/download-video.md](references/download-video.md) |

## 通用注意事项

- **`agent-browser eval` 必须用 `--stdin` + heredoc 传递 JS**，不能用内联字符串（`eval "多行JS"`），否则换行符/引号会导致 `SyntaxError`：
  ```bash
  # 正确
  agent-browser eval --stdin <<'EVALEOF'
  var x = 1;
  x;
  EVALEOF

  # 错误（多行时会报 SyntaxError）
  agent-browser eval "var x = 1;
  x;"
  ```
- 小红书页面 networkidle 经常超时，`wait 3000` 即可
- 页面结构可能随版本更新变化，失效需重新探查
- 频繁操作可能触发安全验证（扫码），需用户手动完成
- 下载媒体时必须带 `Referer: https://www.xiaohongshu.com/` 请求头
