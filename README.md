# Sparkle Eraser

A browser-based tool that automatically detects the Gemini / Imagen **sparkle logo** in an image and erases it by inpainting the surrounding pixels — then hands you the cleaned image to download.

**Everything runs 100% in your browser.** No server, no uploads, no API keys, no cost. Your image never leaves your device.

## Use it

Open `index.html` in any modern browser, or host it (e.g. GitHub Pages) and visit the page.

Then just **upload an image** — detection and removal run automatically, and the **Download** button gives you the result.

## How it works

1. **Detect** — the bottom band of the image (where the watermark sits) is scanned first, then the corners, then the whole frame as a fallback. For each region a robust median background is estimated and several contrast thresholds are swept to isolate bright, compact blobs.
2. **Verify shape** — each candidate blob is matched (shape IoU) against a reconstruction of the four-pointed sparkle glyph, at both orientations, so text and other bright objects don't trigger a false removal.
3. **Remove** — the masked region is filled with a pyramid pull-push reconstruction followed by diffusion smoothing, so the patch blends into the surrounding pixels.
4. **Download** — exported as a lossless PNG at the image's full native resolution. Only the erased region is changed; every other pixel is the original.

If auto-detect ever misses, the **Fine-tune** panel provides manual brush, sensitivity, and edge-padding controls.

## Video (`video.html`)

Same detection + inpainting engine, applied to **every frame** of a video, with the audio kept.

- **Pipeline:** demux the MP4 (mp4box.js) → decode frames (WebCodecs `VideoDecoder`) → clean each frame → re-encode H.264 (WebCodecs `VideoEncoder`) → mux video + **pass-through audio** into a new MP4 (mp4-muxer). Audio is copied unchanged, so it stays in sync and loses no quality.
- **fps and resolution are preserved** — each output frame carries its source timestamp, so timing is exact.
- The watermark position is usually fixed, so detection runs periodically (default: every 15 frames) and reuses the mask in between for speed. Lower the interval if the logo moves.
- **Chrome / Edge (desktop) required** — the WebCodecs API isn't available in Safari or Firefox yet. Processing is offline (slower than real-time); a progress bar with an ETA is shown.
- The two CDN libraries (`mp4box`, `mp4-muxer`, both MIT) load at runtime, so `video.html` must be served from a web host (Render, GitHub Pages) or opened locally in Chrome — it can't run inside the claude.ai artifact sandbox.

## Notes

- The tool removes the **visible** sparkle only. Invisible provenance watermarks (e.g. SynthID) are unaffected.
- Please use it on images you're permitted to edit.
- Processing is entirely client-side, so very large images use more browser memory and take a little longer.

## Tech

Single self-contained HTML file — vanilla JavaScript, Canvas 2D, no dependencies or build step.
