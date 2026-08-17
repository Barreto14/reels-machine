---
name: reel-editor
description: >-
  Edit vertical Reels/Shorts from scratch in the terminal with ffmpeg — cut and assemble
  scenes from raw clips, crop to 9:16 vertical, generate a cover from a frame, find and
  download the right royalty-free track (Pixabay, no Content ID) and mux the music into the
  video already synced. Use whenever the user asks to "edit a reel", "make the video", "cut
  the clips", "join the videos", "rough cut", "find/download the reel music", "add the track
  to the video", or has drone/phone clips to turn into a Reel. Brand-agnostic: reads brand
  assets (logo, fonts, colors) from brand.config.json.
---

# 🎬 Reels Machine — terminal Reel editor (ffmpeg)

> Edit real video in the terminal: cut scenes, 9:16 crop, cover, find + download music, and
> mux it all. This skill is the repeatable step-by-step. The intelligence (which clip, the
> script, the copy) comes from the model; ffmpeg does the heavy lifting locally. **No paid
> connectors, no cloud rendering — everything runs on the user's machine.**

## ⚙️ Prerequisite: ffmpeg
Install the static binaries once (no admin password needed on macOS):
```bash
BIN="$HOME/.local/bin"; mkdir -p "$BIN"; cd "$BIN"
curl -sL -o ffmpeg.zip "https://evermeet.cx/ffmpeg/getrelease/ffmpeg/zip"
curl -sL -o ffprobe.zip "https://evermeet.cx/ffmpeg/getrelease/ffprobe/zip"
unzip -oq ffmpeg.zip && unzip -oq ffprobe.zip && rm -f *.zip
chmod +x ffmpeg ffprobe && xattr -dr com.apple.quarantine ffmpeg ffprobe
```
Linux/Windows: install via package manager (`apt install ffmpeg` / `winget install ffmpeg`).
Use full paths so it always resolves:
```bash
FF="$HOME/.local/bin/ffmpeg"; FP="$HOME/.local/bin/ffprobe"
```

## 🎨 Brand config (fill this once)
Create `brand.config.json` next to this skill (or point the user's to it). Everything visual
reads from here — swap these values for the target brand:
```json
{
  "brand_name": "Your Brand",
  "handle": "@yourhandle",
  "logo_path": "/abs/path/to/logo-transparent.png",
  "colors": { "cream": "#F2EFE4", "gold": "#E8B85F", "ink": "#16271F", "accent": "#1F5A3C" },
  "fonts": {
    "display": "/abs/path/to/Display-Bold.ttf",
    "text": "/abs/path/to/Text-SemiBold.ttf",
    "label": "/System/Library/Fonts/Supplemental/Arial Bold.ttf"
  },
  "media_root": "/abs/path/to/raw-clips",
  "output_root": "/abs/path/to/reels-output"
}
```
> No config yet? Ask the user for logo, brand color, a font, and the clips folder, then write it.

---

## STEP 1 · Locate the media
Raw clips live under `media_root` (drone/phone/action-cam). List and confirm exact names
(users sometimes pre-name them in reel order, e.g. `01_HOOK.MP4`, `02.MP4`):
```bash
python3 -c "import os; d='<folder>'; [print(repr(f)) for f in sorted(os.listdir(d)) if f.lower().endswith(('.mp4','.mov'))]"
```
⚠️ Paths with spaces/accents — always quote them.

## STEP 2 · See what each clip shows (you can't "watch" video)
Extract a poster per clip and build a labeled contact sheet, then open it (read the image):
```bash
find "<folder>" -maxdepth 1 -iname '*.mp4' -print0 | while IFS= read -r -d '' v; do
  qlmanage -t -s 640 -o "<tmp>" "$v" >/dev/null 2>&1; done   # macOS
# cross-platform fallback: "$FF" -ss 2 -i "$v" -frames:v 1 poster.jpg
```
Assemble a grid (PIL) labeled with each clip name and look at it to spot the good shots.

## STEP 3 · Script (scene by scene + on-screen text)
Order: hook → context → climax/hero shot → close, ~15-18s total. Rules that hold up:
- **Hook in the first 3s** (curiosity/POV) — the most common failure point.
- **Real footage beats AI clips** (fully-AI reels underperform and feel tired).
- Save the **best clip for the climax** (sunset / hero drone move).

## STEP 4 · Pick the best ~3s of each clip
Drone clips run 20-90s — use only the prettiest stretch. Sample frames and look:
```bash
for t in 03 07 11 15 19; do
  "$FF" -v error -ss "$t" -i "<clip>" -frames:v 1 -q:v 3 -vf scale=380:-1 "<tmp>/c_${t}s.jpg" -y 2>/dev/null; done
```
Note the **start second** of each clip's best moment.
> ⚠️ ffprobe duration is sometimes empty on drone MP4s — if so use fixed timestamps.

## STEP 5 · Build the rough cut (cut + 9:16 crop + concat)
For EACH scene: cut, scale, **center-crop to 9:16 vertical**, 30fps, no audio:
```bash
"$FF" -y -v error -ss <start> -i "<clip>" -t <len> \
  -vf "scale=-2:1920,crop=1080:1920,setsar=1" -r 30 -an \
  -c:v libx264 -preset veryfast -crf 20 -pix_fmt yuv420p "<segN>.mp4"
```
Then concat (identical params → concat demuxer, no re-encode):
```bash
printf "file '%s'\n" <seg01.mp4> <seg02.mp4> ... > list.txt
"$FF" -y -v error -f concat -safe 0 -i list.txt -c copy "rough-cut.mp4"
```
> Typical scene lengths: hook 3s · b-roll 2.5s · climax 3.5s · close 2.5s → ~16-17s.
> Center-crop drops the sides — mention the user can reposition per clip in a mobile editor.

## STEP 6 · Check the cut
Extract ~8 frames across the rough cut, build a sheet, look (rhythm, composition, climax
timing). Adjust any scene's start/len and regenerate.

## STEP 7 · Cover (from a video frame)
Grab the climax frame in high quality and render a cover in the brand identity (logo top-center
+ pillar label + serif title in cream + handle), 1080×1920. Read colors/fonts/logo from
`brand.config.json`:
```bash
"$FF" -v error -ss <t_climax> -i "<clip>" -frames:v 1 -q:v 2 "<folder>/cover-frame.jpg" -y
```
Compose with Python/PIL: gradient scrim at the bottom, display font for the title, a short gold
rule, the handle. Keep nothing below ~28px for legibility. Emojis don't render in most TTFs —
use vector arrows/shapes instead of emoji glyphs.

## STEP 8 · Music (find + download + mux)
1. **Where:** `pixabay.com/music/` (free, no login). Style: match the mood; for calm/cinematic
   look for tracks that **build** (piano+strings, swell ~10-12s).
2. **⚠️ Content ID:** avoid tracks marked **"Content ID Registered"** for business accounts
   (claim/mute risk). Uppbeat (free tier) is a safe alternative.
3. **Download it yourself:** open the track page in a browser and extract the CDN URL:
   `document.documentElement.innerHTML.match(/https:\/\/cdn\.pixabay\.com\/(?:download\/)?audio[^"']+/g)`
   → gives `cdn.pixabay.com/download/audio/...mp3`. Then `curl -sL -o track.mp3 "<url>"`.
   In the same check: `/Content ID/i.test(document.body.innerText)` must be **false**.
4. **Find the swell BY DATA (not by ear):** map the track's energy in 5s windows:
   ```bash
   for s in 0 5 10 15 20 25 30 35 40 45 50 55; do
     v=$("$FF" -v info -ss $s -t 5 -i track.mp3 -af volumedetect -f null - 2>&1 | grep mean_volume | awk '{print $5}')
     echo "t=${s}s ${v} dB"; done
   ```
   Where dB jumps (e.g. -13.9 → -10.5) is the swell start. **music_start = swell_time −
   climax_time_in_video** (swell at 10s, video climax at 7.2s → start music at 2.8s).
5. **Mux (synced):** ~80% volume, fade in 0.4s, fade out ~1.2s:
   ```bash
   "$FF" -y -v error -i "video.mp4" -ss <music_start> -i "track.mp3" \
     -filter_complex "[1:a]afade=t=in:st=0:d=0.4,afade=t=out:st=<dur-1.2>:d=1.2,volume=0.8[a]" \
     -map 0:v -map "[a]" -c:v copy -c:a aac -b:a 160k -shortest "reel-final.mp4"
   ```
6. **Delivery: the Reel comes out READY, with the track embedded.** Save the final `.mp4` under
   `output_root/<date>-<slug>/` and keep the raw track (`_track_<name>.mp3`) for reuse.

## STEP 9 · Analyze a finished cut (from a mobile editor, if used)
Verify with ffprobe (1080×1920 · 30fps · **has an audio track?**), extract frames and check the
on-screen text (hook in first 3s, climax aligned, close), and that no text overlaps on transitions.

## STEP 10 · Posting kit
Caption (human voice, first line for search) + pinned comment (CTA → your funnel) + up to 5
hashtags + location + alt text + cover + stories. Save a `caption-and-distribution.md` in the
reel folder.

---

## 🎯 Flow summary
**Locate media → see clips (posters) → script → pick the 3s → rough cut (9:16 + concat) →
check → cover → music (Pixabay, no Content ID, mux) → posting kit.**

Identity: real, calm, premium, strong hook, human copy. Everything runs locally. Save outputs
under `output_root/<date>-<slug>/`.

---
*Reels Machine — created by Brasa · Tráfego & Mídia 🔥 (@brasa.trafego.vix). MIT. Keep the credit.*
