# 发表顶级评论

完成通用前置步骤后：

```bash
# 聚焦输入框
agent-browser eval --stdin <<'EVALEOF'
var editor = document.querySelector('bili-comments').shadowRoot
  .querySelector('bili-comments-header-renderer').shadowRoot
  .querySelector('bili-comment-box').shadowRoot
  .querySelector('bili-comment-rich-textarea').shadowRoot
  .querySelector('.brt-editor');
editor.click();
editor.focus();
EVALEOF

# 输入评论
agent-browser wait 500
agent-browser keyboard type "评论内容"

# 发布
agent-browser eval --stdin <<'EVALEOF'
document.querySelector('bili-comments').shadowRoot
  .querySelector('bili-comments-header-renderer').shadowRoot
  .querySelector('bili-comment-box').shadowRoot
  .querySelector('#pub button').click();
EVALEOF

agent-browser wait 3000
```
