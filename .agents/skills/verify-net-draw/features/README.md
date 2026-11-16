# NetDraw verification map

This directory is the maintained source for verifying the user-facing behavior of NetDraw (`net-draw`). Read this index before driving the app, then follow the matching feature file as your literal recipe.

## Baseline preconditions

- Start an isolated instance with `.agents/skills/verify-net-draw/scripts/control-net-draw start --run <id>` (use a unique `<id>`, shown as `v1` in recipes below).
- Default viewport is `1440×900`, where all toolbar text labels (`Save`, `Open`, `Examples`, `Journey`) and `#hintbar` are visible.
- Fresh `localStorage` seeds the built-in sample diagram (`14` nodes `n1`–`n14`, `14` edges `e1`–`e14`, `4` zones `z1`–`z4`, `0` journey steps, `73` palette items).
- Run `.agents/skills/verify-net-draw/scripts/control-net-draw doctor --run v1` and require `healthy: true` (`servedMatchesDisk: true`, `examplesServed: true`, `browserResponsive: true`).
- Never drive an instance that was not started by the current verification run.

## Driving conventions

- Start every recipe from the fresh baseline state unless its `Preconditions:` specify otherwise.
- Run all browser interactions through `.agents/skills/verify-net-draw/scripts/control-net-draw`. Never mutate `window` globals or `localStorage` directly.
- Treat every command as literal; substitute only `--run <id>` when using a run ID other than `v1`.
- Prefer `--role` + `--name` for named HTML buttons/menus and explicit `data-*` / ID selectors (`g.nodeg[data-node="..."]`, `.pitem[data-type="..."]`, `#pLabel`) for SVG canvas items and icon-only controls.
- Tear down every run with `.agents/skills/verify-net-draw/scripts/control-net-draw stop --run v1`. Cleanup removes `/tmp/net-draw-verify-v1` and leaves `/tmp/net-draw-evidence-v1` intact.

## Proof and skip reporting

- Capture both the action and the resulting state, not only the final screen.
- UI proof requires all three artifacts under `/tmp/net-draw-evidence-<id>/`: a PNG `screenshot`, an ARIA `snapshot-aria` text dump, and a read-only `state` JSON snapshot.
- Persistence proof requires confirming `localStorage["netdraw.doc.v1"]` or surviving a full `reload`.
- Export/download proof requires `wait-download` plus inspecting the real file bytes/contents on disk inside `/tmp/net-draw-evidence-<id>/downloads/`.
- Record the sub-feature ID and entry point used with every proof artifact.
- Report any unreachable entry point as a failure with the attempted command and unmet precondition. Never mark a skipped entry point as verified via a different path.

## Feature entry contract

Each feature file starts with an H1 title and one paragraph describing the user-visible behavior. It then uses exactly four H2 sections in this order:

1. `Sub-features` lists short IDs with one line for each behavior.
2. `How to get to it (user POV)` lists every user entry point.
3. `Driving it with control-net-draw` starts with `Preconditions:` and uses labeled bullets pairing each user action with an exact command and observable result.
4. `Gotchas` lists traps that can waste or invalidate a verification run.

## Features

- [Nodes and properties](./nodes-and-properties.md) covers palette click/drag placement, inline and panel renaming, device/STIX metadata, disposition tags, status effects, duplicate/delete, undo/redo, and reload persistence.
- [Connections and containers](./connections-and-containers.md) covers click-click and port-drag linking, connection styling (including sneakernet footsteps), network zones, and cross-functional swimlanes.
- [Examples and JSON I/O](./examples-and-json-io.md) covers the bundled `Examples` gallery, `Save` (`Ctrl+S`) JSON export, `Open` JSON import (valid and invalid files), and `Clear canvas` confirmation/undo.
- [Guided journeys](./guided-journeys.md) covers authoring walkthrough steps, canvas object picking, presentation playback, step reveal/dimming, camera focus, and keyboard navigation.
- [Export and recording](./export-and-recording.md) covers `PNG`, `SVG`, and animated `GIF` resolution presets/validation/downloads, plus live canvas `Rec` video capture.
- [Themes](./themes.md) covers switching between `Midnight`, `ING`, `Blueprint`, and `Phosphor` via `#btnTheme`, reload persistence, and theme-matched SVG/PNG exports.
