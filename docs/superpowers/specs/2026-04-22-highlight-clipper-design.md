# Highlight Clipper — Design Spec

Date: 2026-04-22

## Overview

A single-page frontend tool (`highlight-clipper.html`) that takes a video file, automatically detects high-energy "highlight" moments via audio analysis, lets the user review and select clips, edit subtitles, add BGM, then generates and downloads MP4 clips — all in-browser, no server required.

Fits into the existing toolbox pattern: standalone HTML file, `file://` compatible, ffmpeg.wasm for media processing.

---

## User Flow

```
1. Upload video (drag & drop or file picker)
2. Auto-analyze (Web Audio API decodes audio, detects peaks) — progress bar shown
3. Candidate clips displayed as cards
4. User reviews cards: check/uncheck, edit subtitles
5. Global settings: upload BGM, adjust original/BGM volume ratio
6. Click "生成选中片段"
7. Each clip processes sequentially; when done, a "下载" button appears on its card
```

---

## Audio Analysis (Highlight Detection)

- Decode video audio via `AudioContext.decodeAudioData`
- Compute RMS energy in sliding windows (window: ~1s, step: ~0.1s)
- Smooth the energy curve (moving average)
- Find local maxima above a threshold (mean + 1.5× std dev)
- Expand each peak into a clip: extend ±N seconds (default: peak center ±10s, min clip 5s, max 60s)
- Merge overlapping clips
- Sort by energy score descending
- Output: `[{ start, end, score }]`

---

## Clip Cards UI

Each candidate clip shows:
- Timestamp range (e.g. `01:23 – 01:45`)
- Energy score bar
- Checkbox (default: checked for top 5, unchecked for rest)
- Subtitle textarea (pre-filled from extracted subtitle track, editable)

---

## Subtitle Extraction

- ffmpeg.wasm extracts embedded subtitle stream (SRT or ASS) on video load
- Parse into `[{ start, end, text }]` entries
- For each clip, filter entries whose time range overlaps the clip
- Concatenate matched lines (joined by `\n`) into the clip's subtitle textarea
- If no subtitle track exists: textarea is empty, user can type freely
- On generate: burn subtitles via ffmpeg `subtitles` filter if SRT file provided, or `drawtext` for plain text (white, black outline, bottom-center); multi-line text joined with literal `\n` for drawtext

---

## BGM Mixing

- User uploads an audio file (MP3/AAC/WAV)
- Two sliders: original audio volume (default 80%) and BGM volume (default 40%)
- ffmpeg `amix` filter blends the two streams
- BGM is looped or trimmed to match clip duration

---

## Video Generation (ffmpeg.wasm Worker)

Processing per selected clip:

```
ffmpeg -ss {start} -to {end} -i input.mp4   \
       -i bgm.mp3                            \
       -filter_complex "
         [0:a]volume={origVol}[a0];
         [1:a]volume={bgmVol},aloop=loop=-1:size=44100[a1];
         [a0][a1]amix=inputs=2:duration=first[aout];
         drawtext=text='{subtitle}':fontcolor=white:bordercolor=black:borderw=2:x=(w-text_w)/2:y=h-50
       "                                     \
       -map 0:v -map "[aout]"                \
       -c:v libx264 -c:a aac                 \
       output_{n}.mp4
```

- Clips processed one at a time (sequential, not parallel) to avoid memory pressure
- Progress reported back to main thread via `postMessage`
- Output: `Blob` → `URL.createObjectURL` → `<a download>` click triggered on "下载" button

---

## UI Layout

```
┌─────────────────────────────────────┐
│  [拖拽或选择视频文件]                 │
├─────────────────────────────────────┤
│  分析中... ████████░░ 80%            │
├─────────────────────────────────────┤
│  ☑ 01:23–01:45  ████░ 92分          │
│  字幕: [可编辑文本框________________] │
│                                     │
│  ☑ 02:10–02:35  ███░░ 78分          │
│  字幕: [可编辑文本框________________] │
│  ...                                │
├─────────────────────────────────────┤
│  BGM: [上传音频]  原声 ──●── BGM     │
│  [全选] [取消全选]  [生成选中片段]    │
├─────────────────────────────────────┤
│  片段1: ████████ 完成  [下载]        │
│  片段2: ████░░░░ 处理中...           │
└─────────────────────────────────────┘
```

---

## Constraints & Edge Cases

- No subtitle track: textarea empty, user fills manually; drawtext skipped if textarea empty
- No BGM uploaded: skip amix, output original audio only
- ffmpeg.wasm loaded lazily on first generate (not on page load) to avoid blocking
- Large videos (>500MB): analysis may be slow; show spinner with estimated time
- Clip with no audio peaks found: show empty state with message "未检测到明显燃点，请手动调整"

---

## Out of Scope

- Speech-to-text subtitle generation (requires external API)
- Batch upload of multiple source videos
- Cloud storage or sharing
- Video preview player in the tool
