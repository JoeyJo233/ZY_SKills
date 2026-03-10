# 下载小红书无水印视频

## 原理

小红书视频页面的 `<script>` 中包含 `__INITIAL_STATE__` 数据，其中有多个视频流：

| streamDesc 格式 | 含义 | 水印 |
|-----------------|------|------|
| `WM_X264_MP4_*` | H.264 有水印版 | 有 |
| `X265_MP4_WEB_*` | H.265 无水印版 | 无 |

**关键**：选择 `streamDesc` 不带 `WM_` 前缀的流对应的 `masterUrl` 即为无水印地址。

## 完整流程

```bash
# 1. 打开视频页面
agent-browser open "https://www.xiaohongshu.com/explore/笔记ID?xsec_token=xxx"
agent-browser wait 3000

# 2. 提取无水印视频 URL（从 __INITIAL_STATE__ 中获取不带 WM_ 的流）
agent-browser eval --stdin <<'EVALEOF'
var scripts = document.querySelectorAll('script');
var videoUrl = null;
for (var s of scripts) {
  var text = s.textContent;
  if (text && text.includes('__INITIAL_STATE__')) {
    // 匹配所有 stream 对象：找不带 WM_ 的 streamDesc 对应的 masterUrl
    // 优先取 114 流（无水印 H.265）
    var pattern = /"masterUrl"\s*:\s*"([^"]+_114\.mp4[^"]*)"/;
    var match = text.match(pattern);
    if (match) {
      videoUrl = match[1].replace(/\\u002F/g, '/');
      break;
    }
    // 备选：取 115 流
    pattern = /"masterUrl"\s*:\s*"([^"]+_115\.mp4[^"]*)"/;
    match = text.match(pattern);
    if (match) {
      videoUrl = match[1].replace(/\\u002F/g, '/');
      break;
    }
  }
}
videoUrl || 'not found';
EVALEOF

# 3. 用 curl 下载（将上一步输出的 URL 替换进来）
curl -L -o 输出文件名.mp4 "视频URL" -H "Referer: https://www.xiaohongshu.com/"
```

## 注意

- **`eval` 必须用 `--stdin` + heredoc**，不能用内联字符串，否则换行符会导致 `SyntaxError`
- `_259.mp4` 结尾的是有水印版（WM_X264），不要用
- `_114.mp4` 或 `_115.mp4` 结尾的是无水印版（X265）
- H.265 编码需要支持 HEVC 的播放器（macOS 自带播放器支持）
- 下载时必须带 `Referer: https://www.xiaohongshu.com/` 请求头
