# 发表评论

小红书评论区无 Shadow DOM，可直接用标准选择器操作。

## 完整流程

```bash
# 1. 打开帖子页面
agent-browser open "https://www.xiaohongshu.com/explore/笔记ID"
agent-browser wait 3000

# 2. 点击评论输入区域展开评论框
agent-browser find text "说点什么" click
agent-browser wait 1000

# 3. 聚焦 contenteditable 输入框并输入
agent-browser eval 'var el = document.querySelector("#content-textarea"); el.click(); el.focus();'
agent-browser wait 500
agent-browser keyboard type "评论内容"

# 4. 点击发送
agent-browser find text "发送" click
agent-browser wait 3000
```

## 要点

- 评论输入框是 `<p id="content-textarea" contenteditable="true">`，必须用 `keyboard type`
- 点击"说点什么"展开后才会出现"发送"按钮
- 发送成功后评论会出现在评论区顶部
