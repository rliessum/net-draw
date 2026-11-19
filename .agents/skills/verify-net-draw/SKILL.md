---
name: verify-net-draw
description: Drive and verify NetDraw (net-draw), the single-page SVG network, threat, and process-flow diagram web editor, end to end. Use whenever proving UI behavior, canvas editing, connections, zones/swimlanes, bundled examples, JSON save/open, guided journeys, or PNG/SVG/GIF/video exports.
---

# Verify NetDraw (`net-draw`)

NetDraw is a zero-dependency, single-page static web application (`index.html` + `examples/*.json`). It has no build step or backend database, but **must** be served over HTTP (`fetch("examples/...")` fails under `file://`) and persists user edits to `localStorage["netdraw.doc.v1"]` after a 300 ms debounce.

Read [`features/README.md`](./features/README.md) first, then follow the matching recipe in [`features/`](./features/).

> **Why `control-net-draw` instead of T3 `preview_snapshot`:** NetDraw's baseline canvas runs continuous CSS animations (`@keyframes dashflow` on animated links and `@keyframes fxpulse` on node effect halos) inside a large inline `<svg>`. T3's `preview_snapshot` times out after 15,000 ms (`PreviewAutomationTimeoutError`) on this surface. `.agents/skills/verify-net-draw/scripts/control-net-draw` drives an isolated headless Chrome instance directly over Chrome DevTools Protocol (CDP) using Node's built-in `http`, `fetch`, and `WebSocket` — requiring zero `npm install` packages.

## Launch

Pick a unique alphanumeric run ID (for example `v1` or `pr-42`) and start an isolated verification instance from the repository root:

```bash
.agents/skills/verify-net-draw/scripts/control-net-draw start --run v1
```

What `start` does and what "ready" means:

1. Spawns a local static HTTP server bound to `127.0.0.1` on an OS-assigned ephemeral port (`--port <n>` optional) serving the checkout root with `Cache-Control: no-store`.
2. Launches headless Chrome (`1440×900` viewport by default) with `--user-data-dir=/tmp/net-draw-verify-v1/chrome-profile` and configures CDP `Browser.setDownloadBehavior` to write downloads into `/tmp/net-draw-evidence-v1/downloads/`.
3. Because every run gets a fresh Chrome user-data-dir and unique origin (`http://127.0.0.1:<port>`), `localStorage["netdraw.doc.v1"]` starts empty and `boot()` deterministically loads `sampleState()`: **14 nodes** (`n1`–`n14`), **14 edges** (`e1`–`e14`), **4 zones** (`z1`–`z4`), **73 palette items**, and **0 journey steps**.
4. Waits until `document.readyState === "complete"`, `document.title === "NetDraw — Network Diagrams"`, `#palette .pitem` count is `73`, and `#nodesLayer g.nodeg` count is `14`.

**Isolation:** Concurrent runs with different `--run <id>` flags share zero ports, zero `localStorage`, zero browser profiles, and zero download directories, and never touch the user's desktop browser session. Never reuse an active `--run <id>` across concurrent agents.

**Teardown:**

```bash
.agents/skills/verify-net-draw/scripts/control-net-draw stop --run v1
```

## Doctor

Run `doctor` before driving an instance or whenever behavior looks unexpected. It is strictly read-only:

```bash
.agents/skills/verify-net-draw/scripts/control-net-draw doctor --run v1
```

`doctor` exits `0` only when all five checks pass:

- `httpProcessOwned`: recorded `httpPid` is alive and `/proc/<httpPid>/cmdline` matches `control-net-draw _serve`.
- `chromeProcessOwned`: recorded `chromePid` is alive and `/proc/<chromePid>/cmdline` owns `/tmp/net-draw-verify-v1/chrome-profile`.
- `httpHealthOwnerMatch` & `servedMatchesDisk`: `GET /__netdraw_verify_health` matches `httpPid` and `repoRoot`, and the SHA-256 of HTTP `GET /` matches `index.html` on disk (catches wrong servers or stale pages).
- `examplesServed`: `GET /examples/stuxnet-journey.json` returns HTTP 200 with valid `nodes` and `edges` arrays.
- `browserResponsive`: CDP evaluates cleanly against the expected origin with `title === "NetDraw — Network Diagrams"` and `paletteItems === 73`.

## Drive

All interactions go through `.agents/skills/verify-net-draw/scripts/control-net-draw`, which dispatches real CDP mouse, keyboard, file-input, and download events.

### Stable handles in NetDraw

- **Toolbar tools:** `#toolSelect` (`title="Select / Move  (V)"`), `#toolConnect` (`title="Connect  (C)"`), `#toolZone` (`title="Draw zone / network segment  (Z)"`), `#toolLane` (`title="Draw swimlane — cross-functional lanes  (L)"`), `#toolPan` (`title="Pan  (H, or hold Space)"`).
- **History, zoom, grid:** `#btnUndo` (`Ctrl+Z`), `#btnRedo` (`Ctrl+Y`), `#btnZoomOut`, `#zoomPct`, `#btnZoomIn`, `#btnFit` (`F`), `#btnGrid`, `#btnSnap`.
- **I/O, journeys, recording, theme, clear:** `#btnPNG` (`PNG`), `#btnSVG` (`SVG`), `#btnGIF` (`GIF`), `#btnJSON` (`Save` / `Ctrl+S`), `#btnOpen` (`Open` -> `#fileInput`), `#btnExamples` (`Examples` -> `#examplesMenu .exitem[data-file="..."]`), `#btnJourney` (`Journey` -> `#journey`), `#btnRec` (`Rec` -> `#recModal`), `#btnTheme` (`#themeMenu .thitem[data-theme="midnight|ing|paper|terminal"]`), `#btnClear` (`title="Clear canvas"`).
- **Palette items:** `.pitem[data-type="<type>"]` inside `#palette` (73 types across 11 groups, e.g. `router`, `firewall`, `webserver`, `database`, `person`, `threatactor`, `ransomware`, `plc`, `decision`). Click to place near canvas center, or drag onto `#canvas`.
- **Canvas SVG objects:**
  - Nodes: `#nodesLayer g.nodeg[data-node="<id>"]` (ports: `[data-port="<id>"]`).
  - Connections: `#edgesLayer g.edge[data-edge="<id>"]` (`control-net-draw click` automatically targets the geometric Bézier midpoint of `path.vis`).
  - Zones & swimlanes: `#zonesLayer g.zoneg[data-zone="<id>"]` (`control-net-draw click` automatically avoids interior nodes occluding the zone center and targets the zone's title label when needed; resize handles when selected: `[data-zhandle="se"]`).
- **Properties panel (`#props`, visible when exactly 1 item is selected):** `#pLabel`, schema fields `#pF_<field>` (`#pF_dev`, `#pF_ip`, `#pF_dns`, `#pF_notes`, `#pF_category`, `#pF_sophistication`, `#pF_motivation`, `#pF_family`, `#pF_campaign`), `.dispchip[data-disp="<key>"]`, `.fxchip[data-fx="<key>"]`, `.sw[data-c="<hex>"]`, `#pStyle`, `#pArrows`, `#pAnim`, `#pOrient`, `#laneAdd`, `#pDelete`.
- **Inline label editor:** `#labelEditor` (opened by `click --double` on a node, connection, zone, or `[data-lane="<i>"]`).

### Common drive invocations

```bash
# Click by role + accessible name or CSS selector
.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --role button --name "Examples"
.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector '.pitem[data-type="router"]'

# Double-click a node to open the floating #labelEditor, then rename
.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector 'g.nodeg[data-node="n3"]' --double
.agents/skills/verify-net-draw/scripts/control-net-draw fill --run v1 --selector '#labelEditor' --value "Border Router" --commit enter

# Drag from palette or drag on canvas (canvas pixel offsets from #canvas top-left)
.agents/skills/verify-net-draw/scripts/control-net-draw drag --run v1 --from-selector '.pitem[data-type="waf"]' --to-canvas 260,320
.agents/skills/verify-net-draw/scripts/control-net-draw drag --run v1 --from-canvas 80,80 --to-canvas 420,320

# Handle native window.confirm() on Clear canvas
.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector '#btnClear' --dialog accept
```

## Evidence

All proof artifacts for `--run <id>` live in `/tmp/net-draw-evidence-<id>/` (override with `--evidence-dir <absPath>` on `start`). Relative paths passed to `--out` resolve inside `/tmp/net-draw-evidence-<id>/`; browser downloads land in `/tmp/net-draw-evidence-<id>/downloads/`.

### Proof standards

1. **Real user path only:** Never call internal globals (`addNode()`, `applyLoadedDoc()`, `exportPNG()`, `state.nodes.push()`, or `localStorage.setItem()`) to set up or prove behavior. Drive toolbar buttons, palette items, keyboard shortcuts, file inputs, and canvas pointer gestures.
2. **Capture action and resulting state:** Record a before/action state and resulting state — a final screenshot alone does not prove the transition occurred.
3. **Three-part UI + state proof:** For every verified feature, save:
   - A viewport PNG via `screenshot --run <id> --out <feature>/<step>.png` showing the NetDraw chrome and canvas result.
   - An accessibility snapshot via `snapshot-aria --run <id> --out <feature>/<step>.aria.txt`.
   - A structured DOM + `localStorage` readback via `state --run <id> --out <feature>/<step>.state.json`.
4. **Verify side effects from a second view:**
   - **Persistence (`localStorage["netdraw.doc.v1"]`):** `saveSoon()` debounces writes by 300 ms. Either run `reload --run <id>` (which waits 450 ms before reloading) and inspect rendered SVG state, or wait >= 350 ms and confirm `state.localStorage` matches the rendered canvas.
   - **Exports & recordings (`downloads/`):** Run `wait-download --run <id> --match <ext>` and verify the actual file written to `/tmp/net-draw-evidence-<id>/downloads/` (parse `.json`, verify PNG signature + IHDR dimensions for `.png`, verify `<svg` root and absence of `.ports`/`.zhandle` editor handles for `.svg`, verify `GIF89a` + `NETSCAPE2.0` loop block for `.gif`, verify non-empty media container bytes for `.webm`/`.mp4`).

## Cleanup

Always tear down every run — including failed or interrupted iterations — with:

```bash
.agents/skills/verify-net-draw/scripts/control-net-draw stop --run v1
```

Cleanup rules enforced by `stop`:

- Reads `chromePid` and `httpPid` from `/tmp/net-draw-verify-v1/instance.json` and verifies `/proc/<pid>/cmdline` ownership before sending `SIGTERM` (escalating to `SIGKILL` only if still owned after 3 s). Never run `pkill chrome`, `killall node`, or kill by process name.
- Removes the scratch directory `/tmp/net-draw-verify-v1/`.
- **Never deletes `/tmp/net-draw-evidence-v1/`:** `stop` walks `/tmp/net-draw-evidence-v1/`, confirms `evidenceDirExists: true`, and returns the full list of `preservedArtifacts` in its JSON output. Confirm your expected proof files appear in `preservedArtifacts` before declaring verification complete.

## Helpers

`.agents/skills/verify-net-draw/scripts/control-net-draw` is the sole helper executable (`--help` prints full usage):

```bash
# Lifecycle & health
.agents/skills/verify-net-draw/scripts/control-net-draw start              --run <id> [--port <n>] [--width 1440] [--height 900] [--evidence-dir <dir>]
.agents/skills/verify-net-draw/scripts/control-net-draw doctor             --run <id>
.agents/skills/verify-net-draw/scripts/control-net-draw stop               --run <id>
.agents/skills/verify-net-draw/scripts/control-net-draw validate-examples  [--repo-root <dir>]

# User interactions (real CDP mouse, keyboard, file-chooser, and download events)
.agents/skills/verify-net-draw/scripts/control-net-draw click         --run <id> (--selector <css> | --role <r> --name <n>) [--nth <i>] [--double] [--shift] [--dialog accept|dismiss]
.agents/skills/verify-net-draw/scripts/control-net-draw drag          --run <id> (--from-selector <css> | --from-canvas x,y) (--to-selector <css> | --to-canvas x,y) [--shift]
.agents/skills/verify-net-draw/scripts/control-net-draw fill          --run <id> (--selector <css> | --role <r> --name <n>) --value <text> [--commit enter|blur|none]
.agents/skills/verify-net-draw/scripts/control-net-draw select        --run <id> --selector <css> --value <optionValue>
.agents/skills/verify-net-draw/scripts/control-net-draw press         --run <id> --key <key> [--ctrl] [--shift] [--alt] [--meta]
.agents/skills/verify-net-draw/scripts/control-net-draw upload        --run <id> [--selector '#fileInput'] --file <path>
.agents/skills/verify-net-draw/scripts/control-net-draw wait-download --run <id> [--match <substr>] [--timeout-ms 15000]
.agents/skills/verify-net-draw/scripts/control-net-draw reload        --run <id> [--wait-save-ms 450]
.agents/skills/verify-net-draw/scripts/control-net-draw resize        --run <id> --width <px> --height <px>

# Read-only observation & proof artifacts
.agents/skills/verify-net-draw/scripts/control-net-draw state         --run <id> [--out <relOrAbs.json>]
.agents/skills/verify-net-draw/scripts/control-net-draw snapshot-aria --run <id> --out <relOrAbs.aria.txt>
.agents/skills/verify-net-draw/scripts/control-net-draw screenshot    --run <id> --out <relOrAbs.png>
```
