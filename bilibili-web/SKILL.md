---
name: bilibili-web
description: 操作B站(Bilibili)网页。当用户要求在B站发评论、回复评论、或其他B站网页操作时触发。
allowed-tools: Bash(agent-browser *)
---

# B站网页操作（agent-browser）

前提：浏览器已登录B站账号。

## 通用前置步骤

```bash
agent-browser open "https://www.bilibili.com/video/BVxxxxxxxxxx"
agent-browser wait --load networkidle
agent-browser scroll down 2000
agent-browser wait 2000
```

## 核心难点

B站评论区使用多层嵌套 Shadow DOM，普通选择器和 snapshot 无法定位，必须用 `agent-browser eval --stdin` 执行 JS 逐层穿透。输入必须用 `keyboard type`（不能用 fill）。

## 功能参考

| 功能 | 参考文档 |
|------|----------|
| 发表顶级评论 | [references/comment.md](references/comment.md) |
| 回复指定用户评论 | [references/reply.md](references/reply.md) |

## 通用注意事项

- Shadow DOM 结构可能随B站版本更新变化，失效需重新探查
- B站有评论频率限制，大量操作会触发风控
- 发送成功后输入框自动清空，`wait 3000` 后截图确认
