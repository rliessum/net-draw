# Guided journeys

Guided journeys let a user author multi-step narrated walkthroughs by picking canvas objects per step and writing captions, then present the diagram step by step with progressive reveal, active highlighting/dimming, automatic camera framing, and button or keyboard playback controls.

## Sub-features

- `journey-empty-guard` blocks playback with a toast when a diagram has zero journey steps.
- `journey-author-pick` adds steps, selects objects on the canvas through `#pickBar`, writes step captions, and reorders/deletes steps.
- `journey-present-controls` plays a journey from step 1 (`▶ Play journey`) or from a specific step (`▶ preview`), advancing and rewinding via buttons (`Next ›`, `‹ Prev`) and keys (`ArrowRight`, `Space`, `ArrowLeft`).
- `journey-reveal-dim-lock` hides objects belonging only to future steps, dims non-highlighted prior objects (`opacity=".13"`), locks canvas editing during presentation, and restores full editability on `✕` or `Escape`.

## How to get to it (user POV)

- Click `Journey` (`#btnJourney`) in the top bar to open or close the full-screen `Journey / narration` modal (`#journey`).
- Inside `#journey`, click `+ Add step` (`#jAdd`) — or click `◎ pick objects on canvas` (`.jpick`) on an existing step — to enter canvas object-picking mode (`#pickBar`), select objects on `#canvas`, and click `Use selection` (`#pickUse` or `Enter`) or `Cancel` (`#pickCancel` or `Escape`).
- Click `▶ Play journey` (`#jPlay`) in the modal footer, or `▶ preview` (`.jplay`) on any step card, to enter presentation mode (`#presentBar`).
- Step through a presentation with `Next ›` (`#pjNext`), `‹ Prev` (`#pjPrev`), `ArrowRight`, `Space`, or `ArrowLeft`, and exit with `✕` (`#pjExit`) or `Escape`.

## Driving it with control-net-draw

Preconditions:

- NetDraw is started via `.agents/skills/verify-net-draw/scripts/control-net-draw start --run v1`.
- `.agents/skills/verify-net-draw/scripts/control-net-draw doctor --run v1` reports `healthy: true` (baseline sample diagram has `0` journey steps).

- **Open Journey modal and verify empty-play guard.** Click `#btnJourney`, then click `#jPlay` before adding steps. Run `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector '#btnJourney'` and `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector '#jPlay'`. `journeyModal.open` is `true` (`stepCount: 0`), `presenting` remains `false`, and toast reads `"Add a step first"`.
- **Add Step 1 and pick canvas objects.** Click `+ Add step` (which immediately opens `#pickBar`), select `n1` (`Internet`) and Shift-click `n2` (`Edge Firewall`), then confirm with `Use selection` and fill Step 1's caption. Run `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector '#jAdd'`, `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector 'g.nodeg[data-node="n1"]'`, `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector 'g.nodeg[data-node="n2"]' --shift`, `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector '#pickUse'`, and `.agents/skills/verify-net-draw/scripts/control-net-draw fill --run v1 --selector '.jstep[data-i="0"] .jnote' --value "Ingress hits the edge firewall." --commit blur`. `journeyModal.stepCount` is `1` with chips `["Internet", "Edge Firewall"]`.
- **Add Step 2 and pick a downstream object.** Add a second step revealing `n3` (`Core Router`). Run `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector '#jAdd'`, `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector 'g.nodeg[data-node="n3"]'`, `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector '#pickUse'`, and `.agents/skills/verify-net-draw/scripts/control-net-draw fill --run v1 --selector '.jstep[data-i="1"] .jnote' --value "Traffic reaches the core router." --commit blur`. `journeyModal.stepCount` is `2`.
- **Enter presentation mode and verify progressive reveal.** Click `▶ Play journey` (`#jPlay`) and capture Step 1 state. Run `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector '#jPlay'` and `.agents/skills/verify-net-draw/scripts/control-net-draw state --run v1 --out journeys/01-step1.state.json`. `presenting` is `true`, `presentBar` shows `counter: "1 / 2"`, `caption: "Ingress hits the edge firewall."`, `prevDisabled: true`, `nextDisabled: false`, and `n3` (`Core Router`, revealed in Step 2) is omitted from rendered `nodes` (`counts.nodes` is `13`, not `14`) while non-highlighted baseline nodes have `dimmed: true`.
- **Advance to Step 2 and verify reveal + dimming.** Advance via `ArrowRight` (or `#pjNext`). Run `.agents/skills/verify-net-draw/scripts/control-net-draw press --run v1 --key ArrowRight`. `presentBar.counter` becomes `"2 / 2"`, `caption` reads `"Traffic reaches the core router."`, `nextDisabled` is `true`, `counts.nodes` becomes `14` (`n3` is now rendered with `dimmed: false`), and Step 1's `n1`/`n2` remain visible with `dimmed: true`.
- **Exit presentation and confirm full diagram restored.** Press `Escape` (or click `#pjExit`). Run `.agents/skills/verify-net-draw/scripts/control-net-draw press --run v1 --key Escape`. `presenting` becomes `false`, `presentBar.open` is `false`, and all `14` nodes render with `dimmed: false`.
- **Proof.** Capture presentation and restored-state artifacts. Run `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector '#btnJourney'`, `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector '#jPlay'`, `.agents/skills/verify-net-draw/scripts/control-net-draw screenshot --run v1 --out journeys/02-presenting.png`, `.agents/skills/verify-net-draw/scripts/control-net-draw snapshot-aria --run v1 --out journeys/02-presenting.aria.txt`, `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector '#pjExit'`, and `.agents/skills/verify-net-draw/scripts/control-net-draw state --run v1 --out journeys/03-exited.state.json`.

## Gotchas

- Clicking `+ Add step` (`#jAdd`) does **not** stay inside `#journey`; it immediately hides `#journey` and enters canvas object-picking mode (`#pickBar`). Always finish or cancel picking (`#pickUse`, `#pickCancel`, `Enter`, or `Escape`) before looking for `.jstep` elements in `#journey`.
- Step narration `.jnote` is a `<textarea>` wired to `change`. Always pass `--commit blur` when filling `.jnote`; `--commit enter` only inserts a newline and leaves `steps[i].note` unsaved if you immediately click a non-blurring target or inspect state.
- Objects never listed in *any* step's `show` array remain visible (dimmed when a step has highlights) throughout the entire presentation; only objects listed in a *later* step's `show` array are hidden on earlier steps.
- In `body.presenting`, CSS hides `#palette` and all top-bar `.tgroup` groups except `#recGroup` (`Rec` / `Stop`), and canvas pointer editing is disabled until exiting presentation mode.
