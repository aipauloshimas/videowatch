# Videowatch

Give Claude **eyes**. Drop a video — a local file or a URL — and Claude watches it, breaks it down, and answers your questions about it.

Frames sampled at the right density are the eyes, a local Whisper transcript is the ears. Together they let Claude watch a clip start to finish, hand you a structured breakdown, then stick around for follow-up questions — zooming into any frame for detail it didn't already have.

## How it works
1. Drop a video file into the chat, or paste a URL (YouTube, TikTok, Instagram, etc.) and ask Claude to watch it.
2. The skill runs a **preflight check** (`scripts/check_env.py`): it reports what's missing, explains what each piece is FOR, and prints the install command for your OS — then asks permission before installing anything.
3. Frames are extracted at a density tiered to the video's length: 4fps for clips ≤10s, 2fps up to 30s, 1fps beyond that.
4. The audio is transcribed **locally** with Whisper — no upload, no API key.
5. Claude reads the frames and the transcript together and saves a structured breakdown next to the video.
6. Ask anything about the video. Claude answers from what it actually saw and heard, and can zoom into any frame or moment on demand for detail the sampled frames missed.

## Setup (free and local, no API keys)
- Install **Python 3.8+**.
- Install **ffmpeg** (includes `ffprobe`):
  - macOS: `brew install ffmpeg`
  - Windows: `winget install Gyan.FFmpeg` (or `choco install ffmpeg`)
  - Linux: `sudo apt install ffmpeg`
- Install the Python dependencies: `pip install -r requirements.txt` (openai-whisper + yt-dlp)

The first transcription downloads the Whisper `base` model (~150MB) once. Everything else runs locally: no account, no key.

Whisper auto-detects the spoken language on its own, so this works on video in any language.

## Install the skill
This is a **Claude Code skill**. Clone it into your skills folder:

```bash
git clone https://github.com/aipauloshimas/videowatch ~/.claude/skills/videowatch
```

(Windows: clone into `C:\Users\<you>\.claude\skills\videowatch`.)

**Important:** fully quit and reopen Claude Code after installing. Closing the window is not enough — the skill won't load otherwise.

## First use
A few ways to kick it off:
- Drop an `.mp4` into the chat and say **"watch this video"**.
- Paste a YouTube or TikTok URL and say **"what happens in this video?"**
- Or just run `/videowatch`.

## Output files
Everything videowatch creates lives next to the source video:

| File | What it is |
|---|---|
| `frames_<video>/` | extracted frames |
| `<video>.srt` | the Whisper transcript |
| `<video> - breakdown.md` | the saved analysis |
| `zoom_*.jpg` | detail frames pulled during Q&A |

## Advanced: Instagram URLs
Instagram blocks anonymous downloads, so a public reel URL alone often won't work. To get past it:
1. Install a browser extension like **"Get cookies.txt LOCALLY"** and export your cookies while logged into Instagram in that browser.
2. Save the export as `cookies.txt` and tell Claude where it is.
3. The skill adds `--cookies` to the download command for `instagram.com` URLs only — those cookies are never sent to any other host.

Or skip this entirely: download the reel yourself and drop the file into the chat instead.

## Troubleshooting
Common snags and the fix:
- **"command not found: whisper" / "yt-dlp"** after `pip install` — pip's `--user` scripts directory isn't on your PATH. Find it with `python -c "import site; print(site.USER_BASE + '\\Scripts')"` on Windows, or add `~/.local/bin` to your PATH on macOS/Linux.
- **Checking whisper on Windows** — never run `whisper --help` (it crashes on cp1252 terminals). Use `python -c "import whisper; print(whisper.__version__)"` instead.
- **Whisper looks stuck on the first run** — that's the one-time ~150MB model download, not a hang.
- **A site's download suddenly breaks** — extractors age fast; run `pip install -U yt-dlp`.
- **Skill doesn't show up** — you didn't fully quit Claude Code after installing. Quit completely and reopen.

## Privacy
Transcription and frame extraction run locally — your audio never goes to any third-party service. When Claude analyzes the video, the extracted frames and transcript are sent to Anthropic as part of the normal Claude Code session, the same as any other chat. `yt-dlp` contacts only the URL you give it.

## Files
- `SKILL.md` — the skill (workflow, frame density tiers, breakdown format).
- `scripts/check_env.py` — preflight dependency check (reports what's missing and how to install it).
- `requirements.txt` — the Python dependencies (openai-whisper, yt-dlp).

## License
MIT. See `LICENSE`. Built by [shimas](https://github.com/aipauloshimas).
