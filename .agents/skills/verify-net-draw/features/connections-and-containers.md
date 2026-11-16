# Connections and containers

Connections and containers let a user link nodes via Connect mode or port dragging, style links (including animated traffic and air-gapped sneakernet footsteps), group nodes into network segment zones that move with their contents, and organize cross-functional workflows with horizontal or vertical swimlanes.

## Sub-features

- `edge-connect-click` links two nodes by activating `Connect (C)` and clicking source then destination.
- `edge-connect-port` links two nodes in `Select` mode by dragging a node's connection port (`[data-port]`) onto another node.
- `edge-style-sneakernet` updates connection label, line style (`solid`, `dashed`, `dotted`, `sneakernet`), directional arrows, and traffic animation.
- `zone-draw-keep-contents` draws a network segment zone, renames it, moves enclosed nodes with it, and keeps enclosed nodes when deleting the zone.
- `swimlane-draw-lanes` draws a swimlane container, adds/renames/removes lanes (enforcing the minimum 1-lane guard), and toggles horizontal/vertical orientation.

## How to get to it (user POV)

- Click `#toolConnect` (`title="Connect  (C)"`) or press `c`, then click a source node and click a target node (or drag between them).
- Hover any node in `Select / Move (V)` mode and drag one of its four `.port` handles onto another node.
- Click any connection path on `#canvas` to open `#props` (`Connection`), or double-click a connection to edit its inline label.
- Click `#toolZone` (`title="Draw zone / network segment  (Z)"`) or press `z`, then drag a rectangle larger than `50×40` world px on `#canvas`.
- Click `#toolLane` (`title="Draw swimlane — cross-functional lanes  (L)"`) or press `l`, then drag a container on `#canvas`.

## Driving it with control-net-draw

Preconditions:

- NetDraw is started via `.agents/skills/verify-net-draw/scripts/control-net-draw start --run v1`.
- `.agents/skills/verify-net-draw/scripts/control-net-draw doctor --run v1` reports `healthy: true` with baseline `nodes: 14`, `edges: 14`, `zones: 4`.

- **Connect via toolbar click-click.** Switch to `Connect (C)` and link `n5` (`Web Server`) to `n6` (`PostgreSQL`), which are not directly linked in baseline. Run `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector '#toolConnect'`, `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector 'g.nodeg[data-node="n5"]'`, and `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector 'g.nodeg[data-node="n6"]'`. `counts.edges` becomes `15` and `propsPanel.heading` reads `"Connection"`.
- **Style connection with sneakernet and two-way arrows.** Label the new link and set `sneakernet` + `both` arrows. Run `.agents/skills/verify-net-draw/scripts/control-net-draw fill --run v1 --selector '#pLabel' --value "USB Transfer" --commit enter`, `.agents/skills/verify-net-draw/scripts/control-net-draw select --run v1 --selector '#pStyle' --value "sneakernet"`, and `.agents/skills/verify-net-draw/scripts/control-net-draw select --run v1 --selector '#pArrows' --value "both"`. In `state`, the selected edge has `label: "USB Transfer"`, `sneakernetFootsteps: 2`, `markerStart: true`, and `markerEnd: true`.
- **Connect via port drag.** Return to `Select (V)` and drag a port from `n6` (`PostgreSQL`) to `n7` (`NAS`). Run `.agents/skills/verify-net-draw/scripts/control-net-draw press --run v1 --key v` and `.agents/skills/verify-net-draw/scripts/control-net-draw drag --run v1 --from-selector '[data-port="n6"]' --to-selector 'g.nodeg[data-node="n7"]'`. `counts.edges` becomes `16`.
- **Draw and delete a zone without losing contents.** Activate `Zone (Z)`, drag a zone on canvas, rename the auto-opened `#labelEditor` to `"Staging Segment"`, then delete the zone via `#pDelete`. Run `.agents/skills/verify-net-draw/scripts/control-net-draw press --run v1 --key z`, `.agents/skills/verify-net-draw/scripts/control-net-draw drag --run v1 --from-canvas 60,320 --to-canvas 280,500`, `.agents/skills/verify-net-draw/scripts/control-net-draw fill --run v1 --selector '#labelEditor' --value "Staging Segment" --commit enter`, and `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector '#pDelete'`. `counts.zones` rises `4 -> 5` with `"Staging Segment"` and returns to `4` while `counts.nodes` stays `14`.
- **Draw a swimlane, edit lanes, and switch orientation.** Activate `Swimlane (L)`, drag a swimlane container, title it `"IR Playbook"`, add a fourth lane, and switch orientation to vertical columns. Run `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector '#toolLane'`, `.agents/skills/verify-net-draw/scripts/control-net-draw drag --run v1 --from-canvas 780,60 --to-canvas 1160,340`, `.agents/skills/verify-net-draw/scripts/control-net-draw fill --run v1 --selector '#labelEditor' --value "IR Playbook" --commit enter`, `.agents/skills/verify-net-draw/scripts/control-net-draw click --run v1 --selector '#laneAdd'`, and `.agents/skills/verify-net-draw/scripts/control-net-draw select --run v1 --selector '#pOrient' --value "v"`. `counts.zones` is `5`, and the new swimlane renders title `"IR Playbook"` with lanes `["Lane 1", "Lane 2", "Lane 3", "Lane 4"]`.
- **Proof.** Capture screenshot, ARIA tree, and structured state. Run `.agents/skills/verify-net-draw/scripts/control-net-draw screenshot --run v1 --out containers/01-final.png`, `.agents/skills/verify-net-draw/scripts/control-net-draw snapshot-aria --run v1 --out containers/01-final.aria.txt`, and `.agents/skills/verify-net-draw/scripts/control-net-draw state --run v1 --out containers/01-final.state.json`. Artifacts confirm `16` edges (`"USB Transfer"` with `sneakernetFootsteps: 2`) and `"IR Playbook"` with 4 lanes.

## Gotchas

- `addEdge` silently ignores duplicate links between two nodes that already share a connection in either direction (`a->b` or `b->a`) as well as self-links (`a === b`). Pick an unlinked pair when proving link creation.
- Drawing a zone or swimlane requires dragging more than `50` world px wide and `40` world px tall; smaller drags cancel silently without creating a container.
- Finishing a zone or swimlane drag automatically switches the active tool back to `Select` and immediately opens `#labelEditor` focused on the container's title. Commit or `Escape` `#labelEditor` before pressing single-key shortcuts (`v`, `c`, `z`, `l`), which are suppressed while `#labelEditor` has focus.
- Removing lanes via `.lanedel` stops at 1 lane and displays toast `"A swimlane needs at least one lane"`.
