# NetDraw (`net-draw`)

Single-page, zero-dependency SVG network, threat-modeling, OT/Purdue, and process-flow diagram editor.

## Repository layout

- `index.html` — Entire application (inline CSS, SVG canvas layers `#zonesLayer` / `#edgesLayer` / `#nodesLayer`, 73 palette icon/card types, guided journey presenter, PNG/SVG/GIF/video exporters, and `localStorage["netdraw.doc.v1"]` persistence).
- `examples/*.json` — 17 bundled reference diagrams and guided journeys loaded via `fetch("examples/<file>")`. Every file in `examples/` must match an entry in the `EXAMPLES` array in `index.html`.
- `t3.json` — Checked-in T3 Code project configuration (`Dev Server`, `Verify App`, and `Validate Examples` scripts; default `worktree` thread mode).
- `.agents/skills/verify-net-draw/` (linked at `.claude/skills/verify-net-draw`) — Project-local end-to-end browser verification skill and `control-net-draw` CDP harness.

## Running & verifying

- **Serve over HTTP:** `python3 -m http.server 4173 --bind 127.0.0.1` (`file://` breaks `Examples` loading because `fetch("examples/...")` requires HTTP).
- **Validate bundled diagrams:** `.agents/skills/verify-net-draw/scripts/control-net-draw validate-examples`
- **End-to-end UI verification:** Follow `.agents/skills/verify-net-draw/SKILL.md` and `.agents/skills/verify-net-draw/features/README.md`. Use `control-net-draw` (isolated HTTP port + isolated headless Chrome over CDP) rather than `preview_snapshot`, which times out on NetDraw's continuously animated inline SVG canvas.

## Conventions

- Keep NetDraw zero-dependency: no build step, bundler, or runtime CDN imports inside `index.html`.
- Preserve `version: 1` JSON compatibility (`nodes`, `edges` with `{ id, a, b }`, `zones`, and `journey.steps` with `{ note, show, hi, focus }`) and normalize untrusted JSON inputs in `applyLoadedDoc()`.
