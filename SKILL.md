---
name: dyfetch
description: Download a no-watermark mp4 from a Douyin (抖音) share link. Use when the user shares a `v.douyin.com/...` short link or pasted Douyin share text and asks to download / save / extract / transcribe / 转文字 / 提取视频 / 下载 / 保存.
---

# dyfetch — Douyin no-watermark video downloader

Bypasses yt-dlp's broken path on Douyin (the main-site API now requires cookies) by going through the iesdouyin share page + an iPhone UA to grab the no-watermark mp4.

## When to use
- The user pastes a `v.douyin.com/xxxx` short link, or share text containing a link like `3.89 Copy and open Douyin... https://v.douyin.com/xxx/ ...`
- A direct `iesdouyin.com/share/video/<id>/` link works too
- The user asks to "download / save / transcribe / extract" (including the Chinese equivalents 下载 / 保存 / 转文字 / 提取)

## Tool
- `dyfetch` — resolves the short link → downloads the no-watermark mp4 (720p H.264 + AAC); must be on PATH

## Standard flow
```bash
dyfetch "<short link or share text containing a link>" [output dir]
# → douyin_<item_id>.mp4
```

After downloading, if the user wants a transcript, pick an ASR tool:
- **FunASR** (Chinese SOTA, needs GPU): `pip install funasr`, recommended `paraformer-zh + fsmn-vad + ct-punc`
- **WhisperX** (multilingual + word-level timestamps + speakers): `pip install whisperx`
- **whisper.cpp** (CPU-friendly): https://github.com/ggerganov/whisper.cpp

## Implementation notes (useful when debugging)
1. **Follow the short-link 301**: `curl -I` to get `iesdouyin.com/share/video/<item_id>/`
2. **Fetch the share page**: must use an iPhone Safari UA + `Referer: douyin.com` (going through douyin.com main site requires cookies)
3. **Parse JSON**: extract `window._ROUTER_DATA = {...}</script>` from the HTML using bracket counting (you can't regex `};window` — the JSON is followed by `</script>`, not another window assignment)
4. **Remove the watermark**: take `aweme.snssdk.com/aweme/v1/playwm/?video_id=...` from `play_addr/url_list[0]` and change `playwm` to `play`
5. **Download**: again use the iPhone UA + Referer
6. **Anti-scraping fallback**: when the share-page HTML is < 10KB (the ~2.5KB telemetry stub page) or `_ROUTER_DATA` extraction is empty, switch automatically to `www.douyin.com/aweme/v1/web/aweme/detail/?aweme_id=<id>&device_platform=webapp&aid=6383&channel=channel_pc_web` and pull the no-watermark direct link from `aweme_detail.video.play_addr.url_list` (preferring `douyinvod.com`). This endpoint must use the `www.douyin.com` domain — the same-named `iesdouyin` endpoint returns `blocked`.

## Known failure modes
- yt-dlp reports "Fresh cookies needed" — because it hits the douyin.com main-site API. **Do not fall back to yt-dlp**; stay on the iesdouyin share page.
- If `_ROUTER_DATA` parsing fails, first `curl` the HTML to disk and manually grep `window._ROUTER_DATA` to check whether the delimiter changed.
- **Share page returns the ~2.5KB stub (telemetry script only, no `_ROUTER_DATA`)**: Douyin's probabilistic anti-scraping; the script switches to the detail JSON endpoint automatically (see implementation note 6), no manual intervention needed.
- If the downloaded file is < 100KB, it was most likely blocked by anti-scraping — try a different UA.

## ASR post-processing tips
Chinese ASR commonly produces homophone typos, so scan the transcript by eye afterward. (For example, a Chinese ASR engine may swap acoustically near-identical words such as 锐气 ↔ 婉气 or 通病 ↔ 通便; fix any that obviously break the meaning.)

Re-ordering the transcript by speech structure (claim → sub-points → summary) reads better than pasting it sentence-by-sentence in original order.
