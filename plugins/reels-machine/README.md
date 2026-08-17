# 🎬 Reels Machine — terminal Reel editor (Claude Code plugin)

> **Created by Brasa · Tráfego & Mídia** 🔥 — [@brasa.trafego.vix](https://instagram.com/brasa.trafego.vix)
> Free to use and adapt (MIT). Keep the credit and we're happy. 🌲

Edit vertical Reels/Shorts **from scratch, in the terminal**, with ffmpeg — no paid tools, no
cloud rendering, everything runs on your machine. Cut scenes → crop to 9:16 → branded cover →
find & download royalty-free music (Pixabay, no Content ID) → mux it all in sync. The Reel comes
out **ready to post**, with the track embedded.

Built to run inside **Claude Code**: the plugin gives the step-by-step and the exact ffmpeg
commands; the model supplies the judgement (which clip, the script, the copy).

**Brand-agnostic** — plug in your own logo, fonts and colors via `brand.config.json`.

---

## Prerequisites
1. **Claude Code** installed.
2. **ffmpeg + ffprobe** — install once (macOS, no password):
   ```bash
   BIN="$HOME/.local/bin"; mkdir -p "$BIN"; cd "$BIN"
   curl -sL -o ffmpeg.zip "https://evermeet.cx/ffmpeg/getrelease/ffmpeg/zip"
   curl -sL -o ffprobe.zip "https://evermeet.cx/ffmpeg/getrelease/ffprobe/zip"
   unzip -oq ffmpeg.zip && unzip -oq ffprobe.zip && rm -f *.zip
   chmod +x ffmpeg ffprobe && xattr -dr com.apple.quarantine ffmpeg ffprobe
   ```
   Linux/Windows: `apt install ffmpeg` / `winget install ffmpeg`.
3. **Python 3 + Pillow**: `pip3 install pillow`
4. Your **raw clips** (drone/phone) in a folder.

## Install
1. Unzip `reels-machine-plugin.zip`.
2. Point Claude Code at the plugin folder (the one containing `.claude-plugin/plugin.json`).
   The `reel-editor` skill becomes available.
3. Copy `brand.config.example.json` → `brand.config.json` and fill it with your brand
   (logo, colors, fonts, clips folder, output folder).
4. Test: tell Claude *"edit a reel with the clips in <folder>"*.

## Use
Just talk to Claude:
- *"make a reel from these clips: <folder>"*
- *"find the music and mux it into the video"*
- *"make the cover for this reel"*

Claude follows the skill: locate media → view clips → script → pick the best seconds →
9:16 rough cut → cover → Pixabay track → delivers the finished `.mp4`.

## House rules baked in
- **Real footage beats AI clips.**
- **Hook in the first 3 seconds** decides reach.
- **No "Content ID Registered"** tracks on business accounts (claim/mute risk).
- Reel delivered **with the track embedded**, saved under `output_root/<date>-<slug>/`.

---

## Credits
Made with 🔥 by **Brasa · Tráfego & Mídia** — [@brasa.trafego.vix](https://instagram.com/brasa.trafego.vix).
Licensed MIT: use it, adapt it, ship it. Swap `brand.config.json` and the reels are yours —
just leave this credit line in place. 🌲
