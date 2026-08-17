# 🎬 Reels Machine

**Edit vertical Reels from scratch — in the terminal.**
No paid editor, no cloud rendering, no subscription. Everything runs on your machine.

Made by [**Brasa · Tráfego & Mídia**](https://instagram.com/brasa.trafego.vix) 🔥 · MIT

---

## Why this exists

I hate editing video.

I run a small traffic-and-media agency and I post Reels every week. Learning yet another
timeline editor was not going to happen, and hiring an editor for a business that was
60 days old was not on the table either.

So I built the thing that does it: a Claude Code plugin that turns raw clips into a finished,
posted-ready `.mp4` — cut, cropped, covered, scored and synced — by talking to it in plain
language.

The reels on [@brasa.trafego.vix](https://instagram.com/brasa.trafego.vix) and on our
countryside-inn account are edited with this. It is not a demo.

---

## What it does

```
raw clips  →  pick the best seconds  →  9:16 rough cut  →  branded cover
           →  royalty-free track (Pixabay, no Content ID)  →  muxed, in sync
           →  finished .mp4
```

You talk, it edits:

> *"make a reel from the clips in ~/Movies/drone-june"*
> *"find the music and mux it into the video"*
> *"make the cover for this reel"*

The plugin supplies the repeatable step-by-step and the exact ffmpeg commands.
Claude supplies the judgement — which clip, which three seconds, the script, the copy.

**Brand-agnostic.** Your logo, your fonts, your colors, via `brand.config.json`.

---

## Install

```
/plugin marketplace add Barreto14/reels-machine
/plugin install reels-machine@brasa-tools
```

Then copy `plugins/reels-machine/brand.config.example.json` to `brand.config.json`
and fill in your brand.

### Requirements
| | |
|---|---|
| Claude Code | installed |
| ffmpeg + ffprobe | see below |
| Python 3 + Pillow | `pip3 install pillow` |

**ffmpeg on macOS** (static binaries, no admin password):
```bash
BIN="$HOME/.local/bin"; mkdir -p "$BIN"; cd "$BIN"
curl -sL -o ffmpeg.zip "https://evermeet.cx/ffmpeg/getrelease/ffmpeg/zip"
curl -sL -o ffprobe.zip "https://evermeet.cx/ffmpeg/getrelease/ffprobe/zip"
unzip -oq ffmpeg.zip && unzip -oq ffprobe.zip && rm -f *.zip
chmod +x ffmpeg ffprobe && xattr -dr com.apple.quarantine ffmpeg ffprobe
```
Linux: `apt install ffmpeg` · Windows: `winget install ffmpeg`

---

## House rules baked in

These are not opinions — they are what we measured on our own accounts:

- **Real footage beats AI clips.** A reel we cut from generated clips underperformed badly:
  65,8% of viewers dropped in the first 2 seconds.
- **The hook lives in the first 3 seconds.** It decides reach. Everything else is delivery.
- **No Content-ID-registered tracks on business accounts** — claim and mute risk.
- The Reel is delivered **with the track already embedded**, saved under
  `output_root/<date>-<slug>/`.

---

## License

MIT — use it, adapt it, ship it, sell what you make with it.
Keep the credit line and we're happy.

Made with 🔥 by **Brasa · Tráfego & Mídia** — [@brasa.trafego.vix](https://instagram.com/brasa.trafego.vix)
