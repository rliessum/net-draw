# Export and recording

Export and recording let a user render the diagram cleanly (without editor ports or resize handles) to downloadable `PNG`, `SVG`, or seamless looping animated `GIF` files at preset or custom resolutions, and capture live interactive canvas presentations to downloadable `.webm` / `.mp4` video.

## Sub-features

- `export-svg` exports a vector `.svg` file at preset or custom dimensions with editor handles stripped.
- `export-png` rasterizes and downloads a `.png` file matching the chosen resolution preset or custom pixel box.
- `export-gif` encodes and downloads a 20-frame looping `GIF89a` animation at a selected width preset.
- `export-validation-empty` enforces minimum/maximum pixel caps in `#expModal` and blocks export on an empty canvas with a toast.
- `record-video` records live canvas frames via `Rec` -> `● Start recording` -> `Stop` and downloads a `.webm` or `.mp4` video file.

## How to get to it (user POV)

- Click `PNG` (`#btnPNG`), `SVG` (`#btnSVG`), or `GIF` (`#btnGIF`) in the top bar to open the `Export` resolution dialog (`#expModal`), choose a preset chip (`.expchip`) or `Custom…` (`#expW` × `#expH` with `lock ratio`), and click `Export` (`#expGo`).
- Click `Rec` (`#btnRec`) in the top bar to open `Record diagram` (`#recModal`), choose format and audio (`No audio`, `Microphone narration`, or `Music file`), click `● Start recording` (`#recStart`), interact with or present the canvas, and click `Stop` (`#btnRec`) to download the video.

## Driving it with control-net-draw

Preconditions:

- NetDraw is started via `.agents/skills/verify-net-draw/scripts/control-net-draw start --run v1`.
- `.agents/skills/verify-net-draw/scripts/control-net-draw doctor --run v1` reports `healthy: true` with baseline `nodes: 14`.

- **Export SVG at 1× preset.** Open `SVG` export and confirm. Run `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector '#btnSVG'`, `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector '#expModal .expchip[data-id="1x"]'`, `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector '#expGo'`, and `.agents/skills/verify-net-draw/scripts/control-net-draw wait-download --run v1 --match .svg`. Toast starts with `"SVG exported"`, and the downloaded `network-diagram-<W>x<H>.svg` in `/tmp/net-draw-evidence-v1/downloads/` starts with `<svg`, includes `Edge Firewall`, and contains zero `.ports` or `.zhandle` elements.
- **Validate custom size bounds and export PNG.** Open `PNG` export, choose `Custom…`, verify `< 16 px` disables `#expGo`, then set `1200` width (with `lock ratio` checked) and export. Run `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector '#btnPNG'`, `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector '#expModal .expchip[data-id="custom"]'`, `.agents/skills/verify-net-draw/scripts/control-net-draw fill --run v1 --selector '#expW' --value "8" --commit blur` (confirm `exportModal.invalid: true` and `exportModal.exportDisabled: true` via `state`), `.agents/skills/verify-net-draw/scripts/control-net-draw fill --run v1 --selector '#expW' --value "1200" --commit blur`, `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector '#expGo'`, and `.agents/skills/verify-net-draw/scripts/control-net-draw wait-download --run v1 --match .png`. The downloaded `network-diagram-1200x<H>.png` has the 8-byte PNG signature (`89 50 4e 47 0d 0a 1a 0a`) and IHDR width `1200`.
- **Export animated GIF at 480 w.** Open `GIF` export, choose `480 w` for fast encoding, and export. Run `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector '#btnGIF'`, `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector '#expModal .expchip[data-id="w480"]'`, `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector '#expGo'`, and `.agents/skills/verify-net-draw/scripts/control-net-draw wait-download --run v1 --match .gif --timeout-ms 20000`. The downloaded `network-diagram-480x<H>.gif` begins with `GIF89a` and contains the `NETSCAPE2.0` infinite-loop extension.
- **Record live canvas video.** Open `Rec`, start recording with default `No audio`, pan/zoom the canvas briefly so frames advance, then click `#btnRec` (`Stop`) to save. Run `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector '#btnRec'`, `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector '#recStart'` (confirm `recordingActive: true` and `recButtonText` starts with `"Stop"`), `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector '#btnZoomIn'`, `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector '#btnFit'`, `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector '#btnRec'`, and `.agents/skills/verify-net-draw/scripts/control-net-draw wait-download --run v1 --match network-diagram-recording`. `recordingActive` returns to `false` and `network-diagram-recording.webm` (or `.mp4`) exists in `/tmp/net-draw-evidence-v1/downloads/` with non-zero size.
- **Empty-canvas export guard.** Clear the canvas and click `PNG`. Run `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector '#btnClear' --dialog accept` and `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector '#btnPNG'`. `exportModal.open` stays `false` and toast reads `"Nothing to export — canvas is empty"`. Restore with `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector '#btnUndo'`.
- **Proof.** Capture UI screenshot, ARIA snapshot, and state JSON. Run `.agents/skills/verify-net-draw/scripts/control-net-draw screenshot --run v1 --out export/01-final.png`, `.agents/skills/verify-net-draw/scripts/control-net-draw snapshot-aria --run v1 --out export/01-final.aria.txt`, and `.agents/skills/verify-net-draw/scripts/control-net-draw state --run v1 --out export/01-final.state.json`. Confirm all four downloaded files (`.svg`, `.png`, `.gif`, and `network-diagram-recording.*`) plus UI proof files survive in `/tmp/net-draw-evidence-v1/`.

## Gotchas

- Pressing `Enter` inside `#expW` or `#expH` immediately invokes `runExport()` and closes `#expModal` when the size is valid! Use `--commit blur` (not `--commit enter`) when filling `#expW` / `#expH` if you want to inspect `#expOut` or adjust `#expH` before exporting.
- `GIF` export renders and LZW-encodes 20 frames on the main thread (`btnGIF.disabled = true` while `gifBusy` is active). Use `w480` or `w640` for verification runs so encoding completes in ~1–3 seconds instead of 15+ seconds at `1920 w`.
- Choosing `Microphone narration` in `#recModal` on a headless host without audio capture hardware falls back gracefully to video-only recording with toast `"Microphone unavailable (...) — recording without audio"`. Use default `No audio` unless specifically testing audio fallback or `--selector '#recMusic'` upload.
