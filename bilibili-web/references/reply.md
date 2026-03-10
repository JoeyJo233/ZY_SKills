# 回复指定用户评论

完成通用前置步骤后：

```bash
# 1. 找到目标用户评论并点击回复按钮
agent-browser eval --stdin <<'EVALEOF'
var feed = document.querySelector('bili-comments').shadowRoot.querySelector('#feed');
var threads = feed.querySelectorAll('bili-comment-thread-renderer');
var targetIndex = -1;
for (var i = 0; i < threads.length; i++) {
  var userInfo = threads[i].shadowRoot.querySelector('bili-comment-renderer')
    .shadowRoot.querySelector('bili-comment-user-info');
  var name = userInfo.shadowRoot.querySelector('.user-name') || userInfo.shadowRoot.querySelector('a');
  if (name && name.textContent.trim() === '目标用户名') { targetIndex = i; break; }
}
var renderer = threads[targetIndex].shadowRoot.querySelector('bili-comment-renderer');
renderer.shadowRoot.querySelector('bili-comment-action-buttons-renderer')
  .shadowRoot.querySelector('#reply button').click();
targetIndex;
EVALEOF

# 2. 聚焦回复输入框
agent-browser wait 1000
agent-browser eval --stdin <<'EVALEOF'
var thread = document.querySelector('bili-comments').shadowRoot
  .querySelector('#feed').querySelectorAll('bili-comment-thread-renderer')[目标索引];
var editor = thread.shadowRoot.querySelector('#reply-container bili-comment-box')
  .shadowRoot.querySelector('bili-comment-rich-textarea').shadowRoot
  .querySelector('.brt-editor');
editor.click();
editor.focus();
EVALEOF

# 3. 输入并发布
agent-browser wait 500
agent-browser keyboard type "回复内容"
agent-browser eval --stdin <<'EVALEOF'
var thread = document.querySelector('bili-comments').shadowRoot
  .querySelector('#feed').querySelectorAll('bili-comment-thread-renderer')[目标索引];
thread.shadowRoot.querySelector('#reply-container bili-comment-box')
  .shadowRoot.querySelector('#pub button').click();
EVALEOF

agent-browser wait 3000
```

## 关键路径

- 回复按钮：`thread.shadowRoot > bili-comment-renderer.shadowRoot > bili-comment-action-buttons-renderer.shadowRoot > #reply button`
- 回复输入框：`thread.shadowRoot > #reply-container > bili-comment-box`（之后结构同顶级评论框）
