---
name: dyfetch
description: Download a no-watermark mp4 from a Douyin (抖音) share link. Use when the user shares a `v.douyin.com/...` short link or pasted Douyin share text and asks to download / save / extract / 转文字 / 提取视频 / 下载 / 保存.
---

# dyfetch — 抖音无水印视频下载

绕过 yt-dlp 在抖音的失效路径(主站接口要 cookies),走 iesdouyin 分享页 + iPhone UA 拿无水印 mp4。

## 何时用
- 用户贴 `v.douyin.com/xxxx` 短链,或带分享文案的 `3.89 复制打开抖音... https://v.douyin.com/xxx/ ...` 文本
- 用户给 `iesdouyin.com/share/video/<id>/` 也行
- 用户说 "下载 / 保存 / 转文字 / 提取"

## 工具
- `dyfetch` — 解析短链 → 下载无水印 mp4 (720p H.264 + AAC),需加入 PATH

## 标准流程
```bash
dyfetch "<短链或含链接的分享文本>" [输出目录]
# → douyin_<item_id>.mp4
```

下载后若用户要转写,自选 ASR:
- **FunASR** (中文 SOTA,需 GPU): `pip install funasr`,推荐 `paraformer-zh + fsmn-vad + ct-punc`
- **WhisperX** (多语言 + 词级时间戳 + 说话人): `pip install whisperx`
- **whisper.cpp** (CPU 友好): https://github.com/ggerganov/whisper.cpp

## 实现要点 (debug 时有用)
1. **跟随短链 301**: `curl -I` 拿到 `iesdouyin.com/share/video/<item_id>/`
2. **抓分享页**: 必须用 iPhone Safari UA + `Referer: douyin.com` (走 douyin.com 主站会要 cookies)
3. **解析 JSON**: HTML 里 `window._ROUTER_DATA = {...}</script>` 用括号计数提取 (不能 regex `};window`,JSON 后面是 `</script>` 不是另一个 window 赋值)
4. **去水印**: 从 `play_addr/url_list[0]` 拿到 `aweme.snssdk.com/aweme/v1/playwm/?video_id=...`,把 `playwm` 改成 `play`
5. **下载**: 同样用 iPhone UA + Referer
6. **反爬 fallback**: 分享页 HTML < 10KB(约 2.5KB 遥测空壳页)或 `_ROUTER_DATA` 提取为空时,自动改走 `www.douyin.com/aweme/v1/web/aweme/detail/?aweme_id=<id>&device_platform=webapp&aid=6383&channel=channel_pc_web`,从 `aweme_detail.video.play_addr.url_list` 取无水印直链(优先 `douyinvod.com`)。该接口必须用 `www.douyin.com` 域名,`iesdouyin` 同名接口返 `blocked`

## 已知失败模式
- yt-dlp 直接报 "Fresh cookies needed" — 因为它走 douyin.com 主站接口。**不要回退到 yt-dlp**,继续走 iesdouyin 分享页就行
- 若 `_ROUTER_DATA` 解析失败,先 `curl` 把 HTML 存下来手动 grep `window._ROUTER_DATA` 看分隔符是否变了
- **分享页返回约 2.5KB 空壳页(只有遥测脚本、无 `_ROUTER_DATA`)**:抖音概率性反爬,脚本会自动转 detail JSON 接口(见实现要点 6),无需手动干预
- 若下载文件 < 100KB,大概率被反爬,换个 UA 再试

## ASR 后处理建议
中文 ASR 常见同音错别字,转写后肉眼扫一遍:
- 锐气 ↔ 婉气、通病 ↔ 通便、教养 ↔ 退养
- 时时 ↔ 试试、咄咄逼人 ↔ 多咄逼人
- 拒绝 ↔ 哪绝、消耗 ↔ 削好

按演讲结构(论点 → 分点 → 总结)重排比按句号顺序贴更可读。
