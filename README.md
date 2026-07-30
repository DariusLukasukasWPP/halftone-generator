# WPP Enterprise Dot Illustration Generator

A browser-based halftone dot illustration tool for WPP Enterprise Solutions. Upload an image or video, tune the dot grid and styling, preview the effect live, and export the result as PNG, SVG, or video.

This project extends the original **[WPP v1 Image Generator](https://my.wpp.com/site/wpp-enterprise-solutions/image-generator)** — a single-page tool for turning still images into branded dot illustrations. It keeps the same visual language and controls, and adds video input, live preview, aspect-ratio and resolution control, and frame-accurate video export.

## Getting started

No install or build step. Everything lives in one HTML file.

```bash
git clone https://github.com/DariusLukasukasWPP/halftone-generator.git
```

Serving over HTTP is recommended rather than opening the file from disk — some browsers refuse to expose video pixels to the canvas on `file://` URLs, which blocks the halftone effect on video:

```bash
npx serve .
```

Then open `http://localhost:3000/WPP_Enterprise_DotIllustratorGenerator.html`.

Still images work fine opened directly from disk. If a video can't be sampled, the app says so and falls back automatically where it can.

## Workflows

### Images

1. Upload an image — click the drop zone or drag a file onto it.
2. Adjust settings.
3. Click **Generate Bitmap**.
4. Download **PNG** or **SVG**.

### Video

1. Upload a video (MP4 H.264 or WebM recommended).
2. Click **Generate & Play** — the halftone effect then previews live as the clip plays.
3. Use the transport bar to play, pause, scrub, and loop.
4. Set **Export Frame Rate** to match your source, then click **Download Video**.

Settings can be changed while a video is paused or playing; the preview repaints immediately.

## Controls

### Preset

Style presets (**Default**, **High Contrast Threshold**) set the grid, tone, jitter, and colour controls in one step. **Reset Settings** returns everything to defaults.

### Sizing & grid

- **Max / Min Circle Size** — dot diameter range in pixels
- **Grid Spacing** — distance between sample points
- **Grid Type** — square or hex (offset rows)

### Tone & mapping

- **Render Mode** — continuous tone or binary threshold
- **Invert Brightness** — swaps which tones are emphasised, for dark-background sources
- **Contrast**, **Threshold** (threshold mode only), **Opacity Variation**

### Variation & jitter

- **Position Jitter** and **Size Variation** — randomised offset and scale per dot

Jitter comes from a hash of each grid cell rather than `Math.random()`, so the pattern is identical every time a given frame is drawn. That's what stops dots crawling between video frames, and it keeps PNG and SVG exports consistent with what's on screen.

### Colour & appearance

- **Colour Mode** — single brand colour or sampled source colours
- **Dot Colour** / **Background Colour** — picker plus hex input

### Output

- **Aspect Ratio** — original, 16:9, 1:1, or 9:16 (centre crop)
- **Output Resolution** — native, 1080p, 1440p, or 2160p, measured on the short edge

Both apply to images and video.

### Video playback & export

- **Preview Frame Rate** — on-screen smoothness only; does not affect the exported file
- **Export Frame Rate** — frame rate of the downloaded video (12–60 fps)
- **Max Frame Size** — caps the longest edge during video *preview* to keep playback responsive
- **Export Bitrate** — 8 / 16 / 32 / 50 Mbps; halftone dots are fine, high-frequency detail that a low bitrate smears together

## Export

| Format | Notes |
| --- | --- |
| **PNG** | Raster snapshot of the current frame |
| **SVG** | Vector output matching the visible frame |
| **MP4 / WebM** | Full clip, rendered frame by frame |

### How video export works

Video export runs **offline** rather than recording the screen in real time. The app steps through the source frame by frame, renders each one, and hands it to a `VideoEncoder` with an explicit presentation timestamp.

This matters because the halftone render can be slower than real time — a 1440p frame at close dot spacing means well over a hundred thousand circles. Under a real-time `MediaRecorder` approach, that made playback stall and baked frozen stretches into the file, which also came out longer than the source. Because every frame now carries its own timestamp, a slow frame only makes the export take longer; it can't affect the timing of the finished video.

Worth knowing:

- Export is **slower than real time** at high resolutions and takes longer than the clip's duration. Progress is shown on the button as a percentage.
- **Cancel** discards the export completely — no partial file is written, so a downloaded video always covers the whole clip.
- There's no visible playback during export. This is expected.
- Source audio is decoded from the original file and encoded to AAC (MP4) or Opus (WebM) where the browser supports it.

MP4/H.264 is preferred for compatibility with QuickTime, Premiere, and After Effects, with VP9/WebM as a fallback.

## Browser support

Offline export needs the **WebCodecs API** (`VideoEncoder`). Where it isn't available the app falls back automatically to the older real-time `MediaRecorder` path, which still produces a file but is subject to the stalling described above on heavy settings.

| Browser | Preview | Offline export |
| --- | --- | --- |
| Chrome / Edge | Yes | Yes |
| Safari | Yes | Recent versions |
| Firefox | Yes | Falls back to `MediaRecorder` |

Image PNG/SVG export works everywhere.

## Known limitations

- **Dot spacing is absolute, not relative.** Grid spacing and dot size are in output pixels, so raising **Output Resolution** produces more, proportionally finer dots rather than the same artwork at higher fidelity — and roughly squares the render cost. Match spacing to the resolution you intend to export at.
- **No automatic source frame-rate detection.** There's no reliable browser API for a video's frame rate, so **Export Frame Rate** is an explicit choice. Set it to your source's rate: exporting below it drops frames, above it duplicates them.
- **Desktop layout only.** The sidebar is a fixed width with no responsive breakpoint, so the tool isn't usable at narrow viewport widths.
- Long or high-resolution exports hold the encoded file in memory before download.

## Project structure

```
halftone-generator/
├── WPP_Enterprise_DotIllustratorGenerator.html   # entire app (HTML, CSS, JS)
└── README.md
```

The single file includes two vendored MIT-licensed libraries, inlined so the app works with no build step and no network access:

- [mp4-muxer](https://github.com/Vanilagy/mp4-muxer) v5.2.2 — © Vanilagy
- [webm-muxer](https://github.com/Vanilagy/webm-muxer) v5.1.4 — © Vanilagy

WebCodecs produces encoded chunks, not a playable container; these assemble those chunks into MP4 and WebM files.

## Development history

- **Initial release** — dot illustration generator for still images (PNG / SVG export)
- **Video support** — video upload, live halftone preview, playback controls, preview FPS, frame downscaling, WebM recording, stable per-frame jitter
- **MP4 recording** — H.264/MP4 preferred with WebM fallback
- **Recording quality** — configurable bitrate and source audio passthrough
- **Presets, invert, aspect ratio & output resolution** — style presets, brightness inversion, centre-crop aspect ratios, and resolution tiers for both images and video
- **Offline frame-accurate export** — replaced real-time canvas recording with WebCodecs frame-by-frame encoding, fixing frozen frames and over-long output; added export cancel, progress reporting, and an export frame rate independent of the preview rate
- **UI fixes** — drag-and-drop upload, standards-mode doctype, clearer source readout

## Credits

Based on the WPP Enterprise Solutions **[Image Generator (v1)](https://my.wpp.com/site/wpp-enterprise-solutions/image-generator)**. Extended with video preview, playback, and export.

**Siv Agerholm** — Motion Designer, WPP — collaborator on coding and implementation.
