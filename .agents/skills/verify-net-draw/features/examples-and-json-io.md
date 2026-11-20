# Examples and JSON I/O

Examples and JSON I/O let a user load any of the 17 bundled reference diagrams and guided journeys over HTTP, export the active diagram to `network-diagram.json`, import a saved `.json` file from disk (rejecting malformed files cleanly), and clear the canvas with confirmation and undo recovery.

## Sub-features

- `examples-load` opens the `Examples` menu, loads a bundled diagram over HTTP, fits the camera, and marks the active item `loaded`.
- `json-save` downloads the current diagram state as `network-diagram.json` via toolbar `Save` or `Ctrl+S`.
- `json-open-valid` imports a NetDraw `.json` file via `#fileInput` and renders its nodes, edges, zones, and journey steps.
- `json-open-invalid` rejects a non-NetDraw file with an error toast while leaving the active diagram intact.
- `canvas-clear-confirm` cancels `Clear canvas` when dismissing `confirm()`, clears when accepting `confirm()`, and restores via `Undo`.

## How to get to it (user POV)

- Click `Examples` (`#btnExamples`) in the top bar and choose a guided journey or static diagram from `#examplesMenu` (or navigate items with `ArrowDown` / `ArrowUp` / `Enter` / `Escape`).
- Click `Save` (`#btnJSON`) or press `Ctrl+S` to download `network-diagram.json`.
- Click `Open` (`#btnOpen`) and choose a `.json` file in the file picker.
- Click `Clear canvas` (`#btnClear`, trash icon at the far right of the top bar) and accept or cancel the browser confirmation dialog.

## Driving it with control-net-draw

Preconditions:

- NetDraw is started via `scripts/control-net-draw start --run v1` (HTTP serving is active so `fetch("examples/...")` succeeds).
- `scripts/control-net-draw doctor --run v1` reports `healthy: true` and `examplesServed: true`.

- **Open Examples menu and load a bundled journey.** Click `Examples` and choose `Stuxnet attack chain` (`stuxnet-journey.json`, 6 steps). Run `scripts/control-net-draw click --run v1 --selector '#btnExamples'` and `scripts/control-net-draw click --run v1 --selector '.exitem[data-file="stuxnet-journey.json"]'`. Toast reads `"Loaded: Stuxnet attack chain — open Journey to play it"`, and reopening `#btnExamples` shows `examplesMenu.loadedFile: "stuxnet-journey.json"` (`17` total `.exitem` entries). Close the menu with `Escape`: `scripts/control-net-draw press --run v1 --key Escape`.
- **Save diagram as JSON.** Click `Save` (`#btnJSON`) and wait for the downloaded file. Run `scripts/control-net-draw click --run v1 --selector '#btnJSON'` and `scripts/control-net-draw wait-download --run v1 --match network-diagram.json`. Toast reads `"Diagram saved as JSON"`, and `/tmp/net-draw-evidence-v1/downloads/network-diagram.json` parses as valid JSON with `nodes`, `edges`, `zones`, and `journey.steps` (length `6`).
- **Dismiss Clear canvas, then accept Clear canvas.** Click `#btnClear` with `--dialog dismiss` first, then `--dialog accept`. Run `scripts/control-net-draw click --run v1 --selector '#btnClear' --dialog dismiss` (canvas remains populated), followed by `scripts/control-net-draw click --run v1 --selector '#btnClear' --dialog accept`. Dialog message `"Clear the entire canvas? (You can still undo.)"` is captured and `counts` becomes `{ nodes: 0, edges: 0, zones: 0 }`.
- **Reject invalid JSON on Open.** Upload `README.md` (not valid JSON) to `#fileInput`. Run `scripts/control-net-draw upload --run v1 --selector '#fileInput' --file README.md`. Toast reads `"Could not read that file — not a NetDraw JSON"` and `counts.nodes` remains `0`.
- **Import saved JSON on Open.** Upload a bundled or just-saved `.json` file (`examples/airgap-sneakernet.json`) via `#fileInput`. Run `scripts/control-net-draw upload --run v1 --selector '#fileInput' --file examples/airgap-sneakernet.json`. Toast reads `"Diagram loaded"`, `counts.nodes` is non-zero, and at least one edge renders `sneakernetFootsteps: 2`.
- **Undo back to cleared state and Redo.** Verify history works across imports/clears. Run `scripts/control-net-draw click --run v1 --selector '#btnUndo'` (`counts.nodes` returns to `0`) and `scripts/control-net-draw click --run v1 --selector '#btnRedo'` (imported air-gap diagram returns).
- **Proof.** Capture UI screenshot, ARIA snapshot, and state JSON. Run `scripts/control-net-draw screenshot --run v1 --out io/01-imported.png`, `scripts/control-net-draw snapshot-aria --run v1 --out io/01-imported.aria.txt`, and `scripts/control-net-draw state --run v1 --out io/01-imported.state.json`. Confirm both `/tmp/net-draw-evidence-v1/downloads/network-diagram.json` and `/tmp/net-draw-evidence-v1/io/01-imported.*` exist.

## Gotchas

- Opening `index.html` over `file://` breaks every item in `Examples` with toast `"Couldn't load example — serve the folder over http, or use Open to pick the file"`. Always launch through `control-net-draw start`.
- At viewport widths `<= 1340px`, CSS hides `.tgroup button .lbl`, so `Save`, `Open`, `Examples`, and `Journey` become icon-only buttons whose accessible names come from their `title` attributes (`"Save diagram as JSON file (Ctrl+S)"`, `"Load a bundled example diagram"`, etc.). Prefer `#btnJSON`, `#btnOpen`, `#btnExamples`, `#btnJourney` or default `1440×900` viewport.
- Clicking `#btnClear` when the canvas is already empty (`nodes: 0`, `edges: 0`, `zones: 0`) is a no-op and does **not** open `confirm()`.
- Always pass `--dialog accept` or `--dialog dismiss` when clicking `#btnClear` on a non-empty canvas; an unhandled native `confirm()` blocks the renderer thread.
