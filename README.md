# Sparkle Eraser

A browser-based tool that automatically detects the Gemini / Imagen **sparkle logo** in an image and erases it by inpainting the surrounding pixels — then hands you the cleaned image to download.

**Everything runs 100% in your browser.** No server, no uploads, no API keys, no cost. Your image never leaves your device.

## Use it

Open `index.html` in any modern browser, or host it (e.g. GitHub Pages) and visit the page.

Then just **upload an image (or several at once)** — detection and removal run automatically, and the **Download** button gives you the result.

## Bulk mode

Drop or select more than one image and the page switches to a grid: each one is processed automatically, with a status dot (removed / not found) on its thumbnail. From there you can:

- Click any card to open it in the full editor — compare before/after, fine-tune, or download it individually — then **← All images** to go back.
- Use the small download icon on a card to grab just that one image without opening it.
- **Download all (ZIP)** to get every cleaned image (including any left untouched because nothing was detected) in one file, at full resolution.
- **Add more images** at any time — they're appended to the same batch.

Zipping is done client-side with [fflate](https://github.com/101arrowz/fflate) (MIT, loaded from a CDN), storing the already-compressed PNGs without re-deflating them.

## How it works

1. **Locate** — the watermark sits at a very consistent relative position (~90% across, ~90% down) regardless of subject, so that's searched first. Because the mark can be extremely low-contrast on busy or bright backgrounds (silk folds, highlights), plain thresholding isn't reliable there: a difference-of-box-blurs matched filter is used instead — a small-radius local average minus a large-radius one, which cancels out slow background gradients while lighting up anything roughly the mark's own size, even when it's too faint to survive a hard brightness cutoff.
2. **Confirm** — a small box centred exactly on that point is checked against a reconstruction of the four-pointed sparkle glyph (shape IoU) and against the local background's hue (a logo is a white icon blended onto whatever's beneath it, so it stays in the *background's* hue family — a differently-colored object, like gold embroidery thread, does not). If nothing is genuinely there, detection falls back to scanning the bottom band, the four corners, and finally the whole frame, for placements outside the usual spot.
3. **Remove** — since the icon's shape is fixed and known, a confirmed detection stamps that exact shape at the confirmed position rather than trying to trace its edges from noisy pixels (which either over-grows on faint backgrounds or under-covers on very low-contrast ones). The stamped region is filled with a pyramid pull-push reconstruction plus diffusion smoothing, and real high-frequency texture is grafted in from an adjacent patch so a woven or grained background doesn't come back as a flat blur.
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
