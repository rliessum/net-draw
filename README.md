# NetDraw

Zero-dependency, single-file SVG editor for network topologies, threat models, OT/Purdue architectures, and process-flow diagrams.

- **Live app:** https://rliessum.github.io/net-draw/

## Features

- **73 palette nodes** across Network, Security & Edge, Servers, Storage, Endpoints, People, Places/Orgs, Industrial/OT, Water & Process, Process Flow, and Threats (with STIX metadata, disposition tags, and animated status effects).
- **Connections & containers:** click-click or port-drag links (solid, dashed, dotted, animated traffic, and air-gapped sneakernet footsteps), movable/resizable network zones, and horizontal/vertical swimlanes with centreline snapping.
- **Themes:** top-bar colour theme selector (`Midnight`, `ING`, `Blueprint`, and `Phosphor`) persisted in `localStorage`, applied to live SVG rendering and all exports.
- **Guided journeys:** author multi-step narrated walkthroughs with progressive reveal, highlighting, and camera framing.
- **Import & export:** 17 bundled examples in `examples/`, JSON save/open, `PNG`/`SVG`/animated `GIF` resolution-controlled exports, and live canvas video recording.

## Run locally

Serve the repository over HTTP (opening `index.html` via `file://` prevents `fetch("examples/...")` from loading bundled diagrams):

```bash
python3 -m http.server 4173 --bind 127.0.0.1
```

Then open `http://127.0.0.1:4173/`.

## Verify locally

Validate all bundled examples against `index.html`:

```bash
scripts/control-net-draw validate-examples
```

Run an isolated headless Chrome health check (requires Node.js and Chrome/Chromium):

```bash
scripts/control-net-draw start --run smoke && scripts/control-net-draw doctor --run smoke && scripts/control-net-draw stop --run smoke
```

## Provenance & Copyright

Imported from [https://mr-r3b00t.github.io/net_draw/](https://mr-r3b00t.github.io/net_draw/) (`© 2026 Xservus Limited`). The upstream copyright notice in `index.html` is preserved verbatim; upstream did not publish an explicit open-source license on the hosted page, so no separate license file is added here.
