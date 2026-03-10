# 回复指定评论

## 完整流程

```bash
# 1. 打开帖子页面
agent-browser open "https://www.xiaohongshu.com/explore/笔记ID"
agent-browser wait 3000

# 2. 找到目标评论并点击回复按钮（通过评论文本匹配）
agent-browser eval --stdin <<'EVALEOF'
var comments = document.querySelectorAll('.parent-comment');
for (var i = 0; i < comments.length; i++) {
  if (comments[i].textContent.includes('目标评论中的关键文本')) {
    comments[i].querySelector('.reply.icon-container').click();
    break;
  }
}
EVALEOF

# 3. 聚焦输入框并输入（与新评论共用同一个 #content-textarea）
agent-browser wait 1000
agent-browser eval 'var el = document.querySelector("#content-textarea"); el.click(); el.focus();'
agent-browser wait 500
agent-browser keyboard type "回复内容"

# 4. 发送
agent-browser find text "发送" click
agent-browser wait 3000
```

## 要点

- 回复按钮是 `.reply.icon-container`，在 `.parent-comment` 内
- 点击回复后，底部会弹出回复框并显示"回复 @用户名"
- 输入框与新评论共用 `#content-textarea`
- 如果评论需要滚动才能看到，先滚动评论区找到目标评论
