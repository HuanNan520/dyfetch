# dyfetch

No-watermark mp4 downloader for Douyin. Bypasses yt-dlp's broken path on Douyin (the main-site API now requires cookies) by going through the iesdouyin share page + an iPhone UA to grab the source directly.

Ships with a [Claude Code](https://docs.claude.com/en/docs/claude-code) SKILL, so pasting a Douyin short link to Claude triggers it automatically.

## Install

Dependencies: `bash` `curl` `python3` `file` (standard Linux tools, available on WSL / macOS).

```bash
git clone https://github.com/HuanNan520/dyfetch.git
cd dyfetch

# 1. Install the script into your PATH
sudo install dyfetch /usr/local/bin/
# Or without sudo: install -D dyfetch ~/.local/bin/dyfetch (make sure ~/.local/bin is on PATH)

# 2. (Optional) Let Claude Code auto-detect Douyin links
mkdir -p ~/.claude/skills/dyfetch/
cp SKILL.md ~/.claude/skills/dyfetch/
```

## Usage

```bash
# Short link
dyfetch https://v.douyin.com/_7K_L3Mu-44/

# Paste the full share text (the link is extracted automatically)
dyfetch "3.89 Copy and open Douyin, check out [someone's video] https://v.douyin.com/xxx/ AbC:/"

# Specify an output directory
dyfetch https://v.douyin.com/xxx/ ~/Downloads
```

Output file: `douyin_<item_id>.mp4` (720p H.264 + AAC)

## Anti-scraping fallback

Douyin applies probabilistic anti-scraping to the `iesdouyin.com/share/video/<id>/` share page: when triggered, it returns only a ~2.5KB telemetry stub page with no `_ROUTER_DATA`, so the primary extraction path fails.

The script handles this with an automatic fallback: when the share-page HTML is abnormally small (< 10KB) or `_ROUTER_DATA` extraction comes back empty, it switches to the `www.douyin.com/aweme/v1/web/aweme/detail/` JSON API and pulls the no-watermark CDN direct link from `aweme_detail.video.play_addr.url_list` (preferring `douyinvod.com`). This API is not on the anti-scraping list and needs no cookie; the whole process is transparent to the user and requires no extra arguments.

> Note: the same-named `iesdouyin.com/aweme/v1/web/aweme/detail/` endpoint returns a `blocked` / encrypted error code — you must use the `www.douyin.com` main-site domain.

## Companion ASR (optional)

If you want a "video → text → feed to an LLM for analysis" workflow after downloading, pick a transcription tool:

| Tool | Best for | Install |
|---|---|---|
| [FunASR](https://github.com/modelscope/FunASR) | Chinese SOTA, punctuation + speaker diarization | `pip install funasr`, recommended models `paraformer-zh + fsmn-vad + ct-punc + cam++` |
| [WhisperX](https://github.com/m-bain/whisperX) | Multilingual, word-level timestamps + speakers | `pip install whisperx` |
| [whisper.cpp](https://github.com/ggerganov/whisper.cpp) | Runs without a GPU | see repo |

## Working with Claude Code

Once SKILL.md is placed in `~/.claude/skills/dyfetch/`, Claude Code auto-detects it on startup. After that, paste a Douyin short link in a conversation and Claude will:

1. Automatically call `dyfetch` to download the no-watermark mp4
2. If you have an ASR tool installed, run it to transcribe
3. Feed the text back to itself to analyze the video / outline it / answer your questions

## Known limitations

- Douyin only. For Bilibili / YouTube, `yt-dlp` works directly — you don't need this.
- If Douyin's share-page HTML structure changes, the `_ROUTER_DATA` parsing may need adjustment — see the "Known failure modes" section in SKILL.md.
- When the share page is blocked by anti-scraping, it falls back to the detail JSON API automatically (see the section above); if both paths fail at once, the UA / API policy has most likely changed again.
- It does not circumvent risk control — it only uses the legitimate share page + a mobile UA. If Douyin blocks mobile access, this approach stops working.

## License

MIT
