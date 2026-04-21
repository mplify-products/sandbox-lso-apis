# Mplify LSO API Partners Infographic — Project Context

## What this project is

A single-page D3.js force-directed graph infographic hosted on GitHub Pages. It visualises interoperability between companies on the Buy and Sell sides of LSO (Lifecycle Service Orchestration) APIs standardised by Mplify Alliance. The data is live-fetched from a Google Sheet. Users can switch between different API infographics via a dropdown.

**Live URL:** https://mplify-products.github.io/lso-apis/ (or similar GitHub Pages URL)  
**Main file:** `index.html` (all HTML, CSS and JS in one file — no build step)  
**GitHub repo:** mplify-products/lso-apis (or danielmef.github.io)

## Files in the repo root

- `index.html` — the entire application
- `Mplify-logo.png` — Mplify Alliance logo (white, used with `filter: brightness(0) invert(1)`)
- `LSO Business and Operational APIs.png` — API map image shown in the left panel with hotspots
- `CLAUDE.md` — this file

## Design system

- **Background:** `#04091A` (near-black navy)
- **Text:** `#E8EDF5`
- **Muted text:** `rgba(232,237,245,0.45)`
- **Buy side colour:** `#3D7BC7` (blue)
- **Sell side colour:** `#FD7801` (orange)
- **Fonts:** `Barlow Condensed` (headings, uppercase) + `DM Sans` (body)
- **No build tools** — pure HTML/CSS/JS, edit and deploy directly

## Google Sheets data source

**Spreadsheet ID:** `1Bu8u4j3JKJ_7K9rN7XNB2q1bs9FJv6tbQK_O31Hr9so`  
**API Key:** `AIzaSyBO2A-xcrPpcj9g0ebdlnDlx1AjvP90xzQ`  
**API Key restriction:** domain-restricted to `mplify-products.github.io`

### Sheet tab name → GID mapping (GID_TO_NAME)

| GID | Tab name | Purpose |
|-----|----------|---------|
| 399938948 | menu | Dropdown config (label, links_gid, main title, subtitle) |
| 1901457844 | meta-nodes | All company nodes with columns per API |
| 1881293424 | lso.av2.links | Address Validation links |
| 1279159249 | lso.poq2.links | Product Offering Qualification links |
| 2008560393 | lso.q4.links | Quote links |
| 930832971 | lso.po5.links | Product Order links |
| 1871851720 | non-lso.links | Non-LSO APIs links |
| 2067951233 | form | Implementation survey form options |
| 1595771123 | badges | Badge name → URL mapping |

### Adding a new API infographic

1. Create a new links sheet tab in the Google Sheet (columns: `source`, `target`)
2. Add its GID and tab name to `GID_TO_NAME` in `index.html`
3. If it uses the meta-nodes column filter, add its GID → column mapping to `LINKS_GID_TO_COL`
4. If it does NOT have a `LINKS_GID_TO_COL` entry, nodes are automatically filtered to only those referenced in the links sheet (fallback behaviour — correct for Non-LSO and future sheets)
5. Add a row to the `menu` sheet tab so it appears in the dropdown

### Node filtering logic

- `meta` type rows → always excluded
- `hub` type rows → always included
- If `LINKS_GID_TO_COL[linksGid]` exists → filter meta-nodes by that column (`yes`/blank)
- Otherwise → include only nodes whose `id` appears in the links data

## Left panel layout

All left-panel elements are inside `#left-panel`, a `position: fixed` flex column at `top: 110px; left: 32px; width: 338px; gap: 12px`. Elements stack automatically — do NOT use individual fixed `top` values for elements inside it.

Order from top to bottom inside `#left-panel`:
1. `#info-frame` — white-bordered box with title, subtitle, partner counts, legend
2. `#api-dropdown-wrap` — "Select infographic" label + `<select>` dropdown
3. `#zoom-wrap` — Infographic zoom slider
4. `#api-map-wrap` — LSO Business & Operational APIs image with hotspots
5. `#info-box` — white text box with usage instructions

Below `#left-panel` (positioned dynamically by JS using `getBoundingClientRect`):
- `#buttons-wrap` — flex column of 3 buttons, `width: 220px`, gap 10px

### Button order (top to bottom)
1. ☰ View Partners (green `#1a6644`)
2. ✚ Share my API status (blue `#3D7BC7`)
3. 📄 Show LSO API versions (purple `#5a3d8a`)

## Drawers

Three drawers, all sliding up from the bottom (`height: 75vh`):

| Drawer | Trigger | ID |
|--------|---------|-----|
| Partner Companies | View Partners button | `#roster-drawer` |
| LSO API Versions | Show LSO API versions button | `#lso-versions-drawer` |
| Implementation Survey | Share my API status button | `#impl-drawer` (slides from right, `width: 770px`) |

All drawers share `#impl-overlay` as the backdrop. Closing any drawer must remove `active` from both the drawer and the overlay. The roster drawer also removes `visible` from `#roster-wrap` on close.

The roster drawer title is set dynamically in JS:
```js
rosterDrawerTitle.textContent = "INTEROPERABLE PARTNERS — " + titleEl.textContent;
```

## Hotspots on the API map image

Hotspots are `<div class="hotspot">` elements with `position: absolute` inside `#api-map-container`. Position is set as `left: X%; top: Y%` percentages of the image dimensions.

**Current hotspots:**

| ID | API | Position | links_gid |
|----|-----|----------|-----------|
| hotspot-av | Address Validation & Site Query | left: 8%, top: 35% | 1881293424 |

**To add a new hotspot:**
1. Add a `<div class="hotspot">` inside `#api-map-container` with the correct `left`/`top` percentages
2. Add a click handler in the `// ── API Map Hotspots` JS section that navigates to `?links=GID`
3. The image is 1519×509px — use those dimensions to calculate percentages from the icon centre

**Approximate icon positions in the source image (for reference):**

| Icon | Approx left% | Approx top% | API |
|------|-------------|------------|-----|
| Address Validation (magnifier+checklist) | 8% | 35% | lso.av2 |
| Product Catalog | 17% | 35% | — |
| Product Offering Qualification | 26% | 35% | lso.poq2 |
| Quote | 37% | 35% | lso.q4 |
| Product Order | 47% | 65% | lso.po5 |
| Billing & Invoice | 58% | 65% | — |
| Invoice | 68% | 65% | — |
| Trouble Ticketing | 79% | 65% | — |

## D3.js force simulation

- Buy-side nodes pulled left (`W * 0.3`), sell-side pulled right (`W * 0.7`)
- Hub nodes radius: 56px; high importance: 42px; medium: 32px; low: 22px
- Hub-to-hub link: dashed white, with arrow marker
- Company links: coloured by group, 0.6 opacity

## Zoom behaviour

- Initial zoom: 50% (`initScale: 0.5`)
- Animates from 30% → 50% on load over 2 seconds
- Slider syncs with `d3.zoom` via `zoomBehavior`
- `window.zoomToNode(nodeId)` zooms to a specific node and dims others

## Deployment

This is a static site — just commit and push `index.html` (and any new image files) to the `main` branch. GitHub Pages serves it automatically. No build or compile step needed.

```bash
git add index.html
git commit -m "describe the change"
git push
```

## Common tasks

**Add a new API to the dropdown:**
- Add GID to `GID_TO_NAME` and (if column-filtered) `LINKS_GID_TO_COL`
- Create the links sheet tab in Google Sheets
- Add a row to the `menu` sheet

**Add a hotspot to the API map:**
- Add `<div class="hotspot">` in `#api-map-container` with correct `left`/`top` %
- Add click handler in `// ── API Map Hotspots` section

**Change left panel spacing:**
- Adjust `gap` on `#left-panel` — do NOT add individual `top` values to child elements

**Add a new bottom drawer:**
- Follow the pattern of `#roster-drawer` or `#lso-versions-drawer`
- Use `#impl-overlay` as the shared backdrop
- Height should be `75vh`, slide from `bottom: -75vh`
- Add a button to `#buttons-wrap`
