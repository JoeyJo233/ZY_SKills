# 下载帖子图片

## 原理

通过 `window.__INITIAL_STATE__` 运行时对象获取当前帖子的完整图片列表，**无需翻动轮播**。

路径：`window.__INITIAL_STATE__.note.noteDetailMap[noteId].note.imageList`

每个元素包含 `urlDefault`（完整图片 URL）、`width`、`height` 等字段。

> **为什么不从 DOM 提取？**
> - swiper 轮播懒加载，未翻到的图片不在 DOM 中，直接提取会遗漏
> - 弹窗模式下 DOM 中还混杂背景页大量缩略图，需要额外过滤
> - 从 JS 运行时对象提取更可靠、更完整

> **为什么不从 `<script>` 标签文本中正则匹配？**
> - `<script>` 中的 `__INITIAL_STATE__` 是序列化的初始值，弹窗模式下 `noteDetailMap` 的 key 是 `undefined`，笔记数据是后续异步加载填入的
> - 必须通过 `window.__INITIAL_STATE__` 访问运行时对象才能拿到完整数据

## 完整流程

```bash
# 1. 打开帖子页面（必须带 xsec_token）
agent-browser open "https://www.xiaohongshu.com/explore/笔记ID?xsec_token=xxx"
agent-browser wait 3000

# 2. 一次性提取所有图片 URL（从运行时 JS 对象）
agent-browser eval --stdin <<'EVALEOF'
var state = window.__INITIAL_STATE__;
var map = state.note.noteDetailMap;
// 取 noteId：弹窗模式下 currentNoteId 是 Vue 响应式对象（Proxy），不能直接用
// 必须从 map 的 keys 中找到真实 noteId 字符串，跳过 'undefined' key
var noteId = Object.keys(map).find(function(k) { return k !== 'undefined'; });
var note = map[noteId].note;
var imageList = note.imageList || [];
var urls = imageList.map(function(img) { return img.urlDefault; });
JSON.stringify({count: urls.length, urls: urls});
EVALEOF

# 3. 逐个下载
curl -L -o image_1.webp "图片URL" -H "Referer: https://www.xiaohongshu.com/"
curl -L -o image_2.webp "图片URL" -H "Referer: https://www.xiaohongshu.com/"
# ... 依次下载所有图片
```

## 要点

- **`eval` 必须用 `--stdin` + heredoc**，不能用内联字符串，否则换行符会导致 `SyntaxError`
- **必须用 `window.__INITIAL_STATE__`（运行时对象）**，不要从 `<script>` 标签文本正则提取
- **弹窗模式下 `currentNoteId` 是 Vue 响应式对象（Proxy）**，直接 JSON.stringify 会循环引用报错，直接作为 key 访问 map 也会得到 `[object Object]`。**始终从 `Object.keys(map)` 中筛选真实 noteId**，不要依赖 `currentNoteId`
- 图片格式通常为 webp
- 下载时必须带 `Referer: https://www.xiaohongshu.com/` 请求头
