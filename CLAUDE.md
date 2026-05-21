# BP Chart Extensions 2 — CLAUDE.md

## Project
Parameter-driven D3 chart extension for Tableau (BP Diagnostic Tool).
Single file: `index.html`. No build step. Deploy = `git push` to GitHub Pages.

## Live URL
https://bengtcj.github.io/b_tableau_extensions_2/index.html

## Test in browser
Open URL directly — falls back to settings dialog and sample data (no Tableau needed).
Use the ⚙ settings dialog to switch chart type.

## Key differences from v1 (b_tableau_extensions)
- Driven by `Selected Indicator` Tableau parameter, not tab/dashboard name
- Chart type auto-resolves from indicator_id via METRIC_CHARTS lookup
- Per-indicator chart overrides persisted in settings as `chartOverrides` JSON
- Period keys are yearmonth strings ("2025-09") derived from Year + Month fields
- Quarterly aggregation: sum or average per indicator (see INDICATOR_AGGREGATION)
- Incomplete quarters (< 3 months present) dropped from quarterly view
- Qtr/Mo pill toggle appears inline for trend-capable indicators
- Negative values supported (vom, svt) — hbar and scale-figma handle zero line
- coffee_general rows filtered out automatically
- NaN rows dropped; zero values kept

## Architecture
- Single file app — all JS/CSS/HTML in index.html
- Config stored in Tableau Extensions Settings API (persists in .twb)
- Chart router: renderChart() dispatches to individual render functions
- buildDesign() returns per-element font/colour specs, rebuilt on each render
- ResizeObserver triggers re-render on container resize

## State variables
- CONFIG           — user settings (worksheet, fields, colours, metricId, chart)
- _chartOverrides  — { metricId: chartType } persisted to settings
- _viewMode        — 'quarterly' | 'monthly' (session only, not persisted)
- _selectedBrand   — currently highlighted brand
- _lastData        — last rendered dataset (used by resize re-render)
- _dashboard       — Tableau dashboard object

## Tableau setup required (on the dashboard side)
1. Parameter: `Selected Indicator` (string, list of indicator_ids)
2. Parameter: `Select Client Brand` (string, list of brand names)
3. Parameter Action: on click of indicator sheet → set `Selected Indicator`
4. Worksheet feeding extension must contain ALL indicators unfiltered
5. Required fields: Brand Name Upper, SUM(Raw Value), Year, Month, indicator_id, category_id

## Indicator IDs and default charts
| ID     | Default chart  | Aggregation | Trend? |
|--------|----------------|-------------|--------|
| tam    | ban            | none        | no     |
| cagr   | ban            | none        | no     |
| mcon   | ban            | none        | no     |
| eqr    | vbar-stacked   | avg         | yes    |
| ebl    | vbar-stacked   | avg         | yes    |
| sstsr  | hbar           | sum         | yes    |
| sov    | area-100       | sum         | yes    |
| bss    | area-100       | sum         | yes    |
| vom    | line-smooth    | sum         | yes    |
| svt    | line-smooth    | avg         | yes    |
| bt     | scale-figma    | avg         | no     |
| nps    | hbar           | avg         | no     |
| sop    | scale-figma    | avg         | no     |
| dvtr   | hbar           | avg         | no     |
| cra    | hbar           | sum         | no     |
| ba     | hbar           | none        | no     |

## Code style rules
- No ?. optional chaining — use ternary (Tableau embedded browser may not support ES2020+)
- No ?? nullish coalescing — use ||
- No arrow functions in D3 event handlers that need `this` — use function()
- All chart sizing uses el.offsetWidth / el.offsetHeight (not vw/vh — breaks in iframes)
- Brand names always text-transform:uppercase + FONT_LIGHT
- Inline styles on chart elements (D3 pattern) — CSS only for shell/dialog

## Deploy
git add index.html && git commit -m "..." && git push

## Anti-overengineering
Minimal changes only. No new abstractions. Single HTML file — keep it that way.
Read all relevant code before editing. Prove concepts in small increments.

## Known issues / future work
- `ba`: multiscale chart blocked until multi-attribute data confirmed in worksheet
- `nps`: arc chart blocked — guard shows "NPS recalculation pending"
- `bss`: Q4 months carry repeated values in current pipeline — flag to data team
- `_viewMode` (Qtr/Mo) is session-only — not persisted between Tableau sessions
