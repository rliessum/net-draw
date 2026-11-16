# Nodes and properties

Node authoring lets a user place network, threat, industrial, and process-flow nodes from the palette onto the canvas, rename them inline or in the properties panel, attach schema-driven metadata, dispositions, and status effects, duplicate or delete selections, undo/redo edits, and persist changes across browser reloads.

## Sub-features

- `node-add-click` places a new node near canvas center when clicking a palette item.
- `node-add-drag` places a new node at the drop coordinates when dragging a palette item onto the canvas.
- `node-rename-inline` renames a node through the floating double-click `#labelEditor` (and cancels cleanly on `Escape`).
- `node-props-metadata` edits label, schema-specific text/dropdown/textarea fields, disposition tag, status effect badges/halo, and accent colour in `#props`.
- `node-dup-delete-history` duplicates (`Ctrl+D`), deletes (`Delete` or `Delete node`), undoes (`Undo` / `Ctrl+Z`), and redoes (`Redo` / `Ctrl+Y`).
- `node-persist-reload` persists node edits to `localStorage` and restores them across a page reload.

## How to get to it (user POV)

- Click any `.pitem` tile in the left palette (`Network`, `Security & Edge`, `Servers`, `Storage & Data`, `Endpoints`, `People & Misc`, `Places & Orgs`, `Industrial / OT`, `Water & Process`, `Process Flow`, `Threats`) to drop a node near canvas center.
- Drag any `.pitem` tile from the left palette onto `#canvas` to place it at a chosen position.
- Click a single node on `#canvas` in `Select / Move (V)` mode to open the floating `#props` inspector on the right.
- Double-click a node on `#canvas` to open the inline `#labelEditor` input over the node.
- Press `Ctrl+D` to duplicate selected nodes, `Delete` or `Backspace` (or click `Delete node` in `#props`) to remove them, and `Ctrl+Z` / `Ctrl+Y` (or toolbar `Undo` / `Redo`) to step through history.

## Driving it with control-net-draw

Preconditions:

- NetDraw is started via `.agents/skills/verify-net-draw/scripts/control-net-draw start --run v1`.
- `.agents/skills/verify-net-draw/scripts/control-net-draw doctor --run v1` reports `healthy: true` with baseline `nodes: 14`.

- **Baseline capture.** Record initial state before editing. Run `.agents/skills/verify-net-draw/scripts/control-net-draw state --run v1 --out nodes/01-baseline.state.json`. `counts.nodes` is `14` and `undoDisabled` is `true`.
- **Add by palette click.** Click `WAF` in the palette. Run `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector '.pitem[data-type="waf"]'`. `counts.nodes` becomes `15`, `propsPanel.open` is `true` with `heading: "WAF"`, and `undoDisabled` becomes `false`.
- **Add by palette drag.** Drag `Threat Actor` from the `Threats` group onto the canvas. Run `.agents/skills/verify-net-draw/scripts/control-net-draw drag --run v1 --from-selector '.pitem[data-type="threatactor"]' --to-canvas 260,440`. `counts.nodes` becomes `16` and `propsPanel.heading` reads `"Threat Actor"`.
- **Edit STIX dropdown, notes, disposition, and effects.** Populate the selected Threat Actor's properties. Run `.agents/skills/verify-net-draw/scripts/control-net-draw fill --run v1 --selector '#pLabel' --value "Sandworm" --commit enter`, `.agents/skills/verify-net-draw/scripts/control-net-draw select --run v1 --selector '#pF_category' --value "nation-state"`, `.agents/skills/verify-net-draw/scripts/control-net-draw fill --run v1 --selector '#pF_notes' --value "Tracked campaign" --commit blur`, `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector '.dispchip[data-disp="adversary"]'`, and `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector '.fxchip[data-fx="exploit"]'`. On the canvas card, label reads `"Sandworm"`, metadata line shows `"nation-state"` (while `notes` stays panel-only), disposition tag reads `"ADVERSARY"`, effect badge includes `"Exploited"`, and `hasHalo` is `true`.
- **Inline double-click rename and Escape cancel.** Double-click baseline node `n3` (`Core Router`) to rename it, then verify `Escape` leaves it unchanged on a second open. Run `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector 'g.nodeg[data-node="n3"]' --double`, `.agents/skills/verify-net-draw/scripts/control-net-draw fill --run v1 --selector '#labelEditor' --value "Transit Router" --commit enter`, `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector 'g.nodeg[data-node="n3"]' --double`, `.agents/skills/verify-net-draw/scripts/control-net-draw fill --run v1 --selector '#labelEditor' --value "Discard Rename" --commit none`, and `.agents/skills/verify-net-draw/scripts/control-net-draw press --run v1 --key Escape`. Node `n3` renders `"Transit Router"` and `labelEditor.open` is `false`.
- **Duplicate, delete, undo, and redo.** Select `n3`, duplicate it, delete the clone, then undo and redo. Run `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector 'g.nodeg[data-node="n3"]'`, `.agents/skills/verify-net-draw/scripts/control-net-draw press --run v1 --key d --ctrl`, `.agents/skills/verify-net-draw/scripts/control-net-draw press --run v1 --key Delete`, `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector '#btnUndo'`, and `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector '#btnRedo'`. `counts.nodes` moves `16 -> 17 -> 16 -> 17 -> 16`, with `redoDisabled: true` after redo.
- **Reload persistence.** Reload the page and confirm edits survived via `localStorage["netdraw.doc.v1"]`. Run `.agents/skills/verify-net-draw/scripts/control-net-draw reload --run v1`. Rendered `counts.nodes` remains `16`, `"Sandworm"` and `"Transit Router"` remain on the canvas, and `state.localStorage.nodes` equals `16`.
- **Proof.** Capture final UI, ARIA, and structured state artifacts. Run `.agents/skills/verify-net-draw/scripts/control-net-draw screenshot --run v1 --out nodes/02-final.png`, `.agents/skills/verify-net-draw/scripts/control-net-draw snapshot-aria --run v1 --out nodes/02-final.aria.txt`, and `.agents/skills/verify-net-draw/scripts/control-net-draw state --run v1 --out nodes/02-final.state.json`. All three artifacts exist under `/tmp/net-draw-evidence-v1/nodes/` and confirm `"Sandworm"` and `"Transit Router"`.

## Gotchas

- Properties panel text inputs (`#pLabel`, `#pF_*`) commit on `change`, not `input`. Always commit `<input>` fields with `Enter` (`--commit enter`) and `<textarea>` fields (`#pF_notes`) with blur (`--commit blur`), because `Enter` inside a textarea inserts a newline without firing `change`.
- Process-flow shapes (`startend`, `process`, `decision`, `dataio`, `document`, `predefined`, `preparation`, `manualinput`, `delay`, `connector`) center their label inside the polygon/path and intentionally suppress metadata lines on the card even when `#pF_owner` is set.
- `#props` only appears when `selection.size === 1`. Multi-selecting (`--shift` or `Ctrl+A`) hides `#props` until a single item is clicked.
- `saveSoon()` debounces `localStorage` writes by 300 ms. `control-net-draw reload` waits 450 ms automatically before reloading; do not pass `--wait-save-ms 0` right after an edit if testing persistence.
