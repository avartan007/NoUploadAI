# Media inventory

Every image and recording in this repository was produced from the running application or from
the runnable companion demos included in `studio/`. None are stock photos or fabricated mockups.

## Application media

| File | Type | Size | Source | Suggested alt text |
|---|---|---|---|---|
| `hero.png` | PNG, 3200×1800 (2× DPR) | 571 KB | Headless Chromium screenshot of the landing page at 1600×900 | "NoUploadAI landing page with chat-with-your-documents headline and zero-upload badge" |
| `landing-tour.gif` | Animated GIF, 800 px wide, 10 fps | 4.1 MB | Real-time screen recording while scrolling the full landing page | "Scrolling tour of the NoUploadAI landing page sections" |
| `launch-film.gif` | Animated GIF, 720 px wide, 10 fps | 6.5 MB | 10-second excerpt of the in-app launch-film player | "Animated excerpt of the NoUploadAI launch film" |
| `launch-film.mp4` | H.264 MP4, 1280×720, 46 s | 1.5 MB | Screen capture of the full in-app launch-film player | "45-second NoUploadAI launch film" |
| `logo.svg` | Vector SVG, 440×96 | 1.1 KB | Hand-authored brand mark matching the app's terracotta cloud-and-slash icon | "NoUploadAI logo" |

### Notes

- `hero.png` is captured at 2× device pixel ratio so it stays crisp in README cards and slide decks.
- The MP4 is a screen recording of the Canvas player, so it is **video-only**. The application's own player generates its soundtrack live with WebAudio; the app can also export a video with audio via `MediaRecorder` where supported.
- The GIF is a short excerpt chosen to show the "Meet NoUploadAI" and "Drop & embed" scenes; the MP4 contains all seven scenes.
- Numbers printed inside the film ("retrieval ms", "tok/s", "embedding count") are illustrative presentation copy in the application itself, not measurements taken during this capture.

## Companion studio media

These files show the standalone demos under `studio/`, not the core RAG application.

| File | Type | Size | Source | Suggested alt text |
|---|---|---|---|---|
| `studio-tools.gif` | Animated GIF, 800 px wide, 10 fps | 1.3 MB | Recording of the tools hub, card hovers and a jump into the compressor | "NoUpload Studio tools hub with compress, scrub and SVG optimize demos" |
| `studio-compress.png` | PNG, 2880×2000 (2× DPR) | 349 KB | Compressor after loading the bundled sample photo (61.8 KB → 9.4 KB, 85% smaller) | "Image compressor showing original and compressed byte sizes" |
| `studio-compress.gif` | Animated GIF, 800 px wide, 10 fps | 815 KB | Recording of quality, width and format changes updating the preview | "Live image compression with quality and width controls" |
| `studio-scrub.png` | PNG, 2880×2000 (2× DPR) | 397 KB | Scrubber after loading a sample photo with real EXIF (Canon EOS R5, timestamp, GPS 37°46'12"N 122°25'18"W) | "Metadata scrubber listing camera and GPS EXIF fields before and after cleaning" |
| `studio-svg.png` | PNG, 2880×2000 (2× DPR) | 334 KB | Optimizer after loading a design-tool-style SVG (1,151 → 747 characters, 35% removed) | "SVG optimizer with live vector preview and size comparison" |

`studio/sample-photo.jpg` is a generated JPEG with deliberately embedded EXIF (camera make/model,
software, capture time and GPS rationals) so the scrubber demo has real metadata to detect and
remove.

## Reproduction

Captures were made with headless Chromium driven by Playwright, then encoded with FFmpeg:

```bash
# serve the built app and the docs folder
npm run build && npx vite preview --port 4173
python3 -m http.server 4174 --directory docs

# palette-based GIF conversion (example)
ffmpeg -i capture.webm \
  -vf "fps=10,scale=800:-1:flags=lanczos,split[s0][s1];[s0]palettegen=max_colors=128[p];[s1][p]paletteuse=dither=bayer" \
  -loop 0 output.gif
```

The source application contains no references to these files, so the visual assets can be replaced
or regenerated without touching the build.
