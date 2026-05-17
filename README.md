# dyfetch

抖音无水印 mp4 下载。绕开 yt-dlp 在抖音的失效路径(主站接口要 cookies),走 iesdouyin 分享页 + iPhone UA 直接拿源。

附带 [Claude Code](https://docs.claude.com/en/docs/claude-code) SKILL,贴抖音短链给 Claude 时会自动调用。

## 安装

依赖:`bash` `curl` `python3` `file`(标准 Linux 工具,WSL / macOS 都有)

```bash
git clone https://github.com/HuanNan520/dyfetch.git
cd dyfetch

# 1. 装脚本到 PATH
sudo install dyfetch /usr/local/bin/
# 或不要 sudo:install -D dyfetch ~/.local/bin/dyfetch (确保 ~/.local/bin 在 PATH)

# 2. (可选) 让 Claude Code 自动识别抖音链接
mkdir -p ~/.claude/skills/dyfetch/
cp SKILL.md ~/.claude/skills/dyfetch/
```

## 用法

```bash
# 短链
dyfetch https://v.douyin.com/_7K_L3Mu-44/

# 带分享文案直接贴(自动提取链接)
dyfetch "3.89 复制打开抖音,看看【xxx 的作品】https://v.douyin.com/xxx/ AbC:/"

# 指定输出目录
dyfetch https://v.douyin.com/xxx/ ~/Downloads
```

输出文件:`douyin_<item_id>.mp4` (720p H.264 + AAC)

## 配套 ASR(可选)

下载后想做"视频 → 文字 → 喂给 LLM 解析"工作流,自选转写工具:

| 工具 | 适用 | 安装 |
|---|---|---|
| [FunASR](https://github.com/modelscope/FunASR) | 中文 SOTA,带标点 + 说话人分离 | `pip install funasr`,推荐模型 `paraformer-zh + fsmn-vad + ct-punc + cam++` |
| [WhisperX](https://github.com/m-bain/whisperX) | 多语言,词级时间戳 + 说话人 | `pip install whisperx` |
| [whisper.cpp](https://github.com/ggerganov/whisper.cpp) | 无 GPU 可跑 | 见仓库 |

## 与 Claude Code 配合

SKILL.md 放进 `~/.claude/skills/dyfetch/` 后,Claude Code 启动会自动识别。之后在对话里贴抖音短链,Claude 会:

1. 自动调 `dyfetch` 下载无水印 mp4
2. 如果你装了 ASR,继续调你的 ASR 转写
3. 把文字喂给自己,解析视频内容 / 列大纲 / 答你的问题

## 已知限制

- 只搞抖音。B站 / YouTube 用 `yt-dlp` 直接 work,不需要这个
- 抖音分享页 HTML 结构变了的话,`_ROUTER_DATA` 解析可能要调,见 SKILL.md "已知失败模式" 节
- 不绕风控,只是用合法分享页 + 移动端 UA;若抖音封移动端访问,这个方案就失效

## License

MIT
