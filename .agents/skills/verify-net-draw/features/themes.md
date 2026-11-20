# Themes

Theme selection lets a user switch the entire NetDraw workspace, SVG diagram canvas, and standalone exports between curated dark and light palettes (`Midnight`, `ING` inspired by https://ing.com/, `Blueprint`, and `Phosphor`), persisting the chosen theme across browser reloads without mutating saved diagram data.

## Sub-features

- `theme-menu-switch` opens `#themeMenu` from `#btnTheme`, navigates options by pointer or keyboard, and applies `ING`, `Blueprint`, `Phosphor`, or `Midnight`.
- `theme-ing-tokens` applies ING brand tokens (`#ff6200` ING Orange accent/CTAs/footsteps, `#f5f5f5` canvas, `#ffffff` cards/panels, `#333333` text, `#052962` navy ink) across UI chrome and live SVG nodes/edges/zones.
- `theme-persist-reload` saves the selected theme in `localStorage["netdraw.theme.v1"]` and restores it cleanly on reload.
- `theme-export-fidelity` exports standalone `.svg` and `.png` diagrams rendered in the active theme's background, card gradient, and typography.

## How to get to it (user POV)

- Click the theme button (`#btnTheme`, showing the live dual-swatch dot and current theme name) in the top bar to open `#themeMenu`.
- Click a theme item (`Midnight`, `ING`, `Blueprint`, `Phosphor`) or navigate `#themeMenu` with `ArrowDown` / `ArrowUp` / `Home` / `End` / `Enter` and dismiss with `Escape` or clicking outside.

## Driving it with control-net-draw

Preconditions:

- NetDraw is started via `scripts/control-net-draw start --run v1`.
- `scripts/control-net-draw doctor --run v1` reports `healthy: true` (fresh baseline starts on `theme: "midnight"`).

- **Open Theme menu and switch to ING.** Click `#btnTheme`, inspect `#themeMenu`, and choose `ING`. Run `scripts/control-net-draw click --run v1 --selector '#btnTheme'`, `scripts/control-net-draw state --run v1 --out themes/01-menu.state.json`, and `scripts/control-net-draw click --run v1 --selector '.thitem[data-theme="ing"]'`. `themeMenu.items` equals `["midnight", "ing", "paper", "terminal"]`; after clicking `ING`, `theme` is `"ing"`, `themeLabel` is `"ING"`, `themeTokens.accent` is `"#ff6200"`, `themeTokens.bg` is `"#f5f5f5"`, `themeTokens.cardStop0` is `"#ffffff"`, and toast reads `"Theme: ING"`.
- **Reload persistence.** Reload the page and confirm `ING` stays active without flashing back to `Midnight`. Run `scripts/control-net-draw reload --run v1`. `state` reports `theme: "ing"`, `savedTheme: "ing"`, and `themeTokens.accent: "#ff6200"`.
- **Theme-faithful SVG export.** Export an SVG while `ING` is active. Run `scripts/control-net-draw click --run v1 --selector '#btnSVG'`, `scripts/control-net-draw click --run v1 --selector '#expGo'`, and `scripts/control-net-draw wait-download --run v1 --match .svg`. The downloaded `.svg` file in `/tmp/net-draw-evidence-v1/downloads/` contains `fill="#f5f5f5"` for the canvas background rect, `stop-color="#ffffff"` in `#cardGrad`, and `fill="#333333"` on node labels.
- **Keyboard navigation and remaining themes.** Open `#btnTheme`, cycle with `ArrowDown` + `Enter`, and test `Escape` dismissal. Run `scripts/control-net-draw click --run v1 --selector '#btnTheme'`, `scripts/control-net-draw press --run v1 --key ArrowDown`, `scripts/control-net-draw press --run v1 --key Enter` (`theme` becomes `"paper"`), `scripts/control-net-draw click --run v1 --selector '#btnTheme'`, and `scripts/control-net-draw press --run v1 --key Escape` (`themeMenu.open` becomes `false`).
- **Proof.** Switch back to `ING`, select `n2` (`Edge Firewall`) so `#props` is open alongside `#themeMenu`, and capture UI, ARIA, and state proof. Run `scripts/control-net-draw click --run v1 --selector '#btnTheme'`, `scripts/control-net-draw click --run v1 --selector '.thitem[data-theme="ing"]'`, `scripts/control-net-draw click --run v1 --selector 'g.nodeg[data-node="n2"]'`, `scripts/control-net-draw screenshot --run v1 --out themes/02-ing.png`, `scripts/control-net-draw snapshot-aria --run v1 --out themes/02-ing.aria.txt`, and `scripts/control-net-draw state --run v1 --out themes/02-ing.state.json`.

## Gotchas

- `themeInk()` adapts bright pastel strokes/text visually on light themes (`ING` and `Blueprint`) at render time without mutating `state.nodes[i].accent` or `state.zones[i].color`, so exported `.json` diagrams remain theme-neutral.
- Opening `#themeMenu` automatically closes `#examplesMenu` (and vice versa).
- At viewport widths `<= 1340px`, `#themeLbl` hides with other `.tgroup button .lbl` spans while `#themeDot` remains visible; target `#btnTheme` or `title^="Switch colour theme"`.
