# AGENTS.md — Chart SVG Template Library

71 SVG templates (charts, diagrams, tables, frameworks). Source of truth: `charts_index.json` (71 entries, each with a selection-rule `summary` — "Pick for X. Skip if Y (use alternative)."). Design rules: `CHART_STYLE_GUIDE.md` (Chinese, 643 lines).

## Entry Points

| What | File |
|---|---|
| Template index | `charts_index.json` — single source of truth, full-read by Strategist |
| SVG design rules | `CHART_STYLE_GUIDE.md` — color (Tailwind Slate palette), typography, shadow filters, gradients, banned features, placeholder conventions, coordinate calibration markers |
| SVG files | 71 `.svg` files named by `key` in index (e.g. `bar_chart.svg`) |
| Coordinate calibration | `svg_position_calculator.py` (in `scripts/`) — BarChartCalculator, PieChartCalculator, LineChartCalculator, RadarChartCalculator, GridLayoutCalculator, SVGPositionValidator |
| Verification workflow | `workflows/verify-charts.md` — run between Executor and post-processing |

## SVG Technical Constraints (from shared-standards.md + CHART_STYLE_GUIDE.md)

**ABSOLUTELY FORBIDDEN:**
- `rgba()` / HSL / named colors — HEX only (`#RRGGBB`)
- `<g opacity="...">` — use `fill-opacity` / `stroke-opacity` on individual elements
- `<image opacity="...">`
- `@font-face` / external fonts / `<style>` tags / `<class>` attributes
- `<feComponentTransfer>` inside `<filter>`

**Allowed conditionally:**
- `<filter>` for shadows — must use `feFlood` primitive (not `feComponentTransfer`)
- Linear/radial gradients (`<linearGradient>`, `<radialGradient>`) — pre-defined in `<defs>`
- `<marker>` for arrowheads — only on `<path>` elements
- `<clipPath>` — only on `<g>` (not individual shapes)

**Fonts:** system only — `Microsoft YaHei`, `SimSun`, `Arial`, `sans-serif`

**Colors:** Tailwind CSS Slate palette for UI elements; data series use theme colors from spec_lock (adapted by Executor)

## Chart Template Structure

Each SVG follows this internal layout:
```
<svg viewBox="0 0 1280 720">
  <defs>...
  <g id="chartArea">        <!-- chart body -->
    <!-- axes, gridlines first -->
    <!-- data elements (bars/lines/slices) second -->
    <!-- chart-plot-area comment marker --> (14 calculator-supported charts)
  </g>
  <g id="chartTitle">       <!-- main title -->
  <g id="legend">           <!-- legend block -->
```

**Coordinate calibration markers** (14 charts with data mapping):
- Rectangular charts (bar, line, scatter, etc.): `<!-- chart-plot-area: x_min,y_min,x_max,y_max -->`
- Polar charts (pie, donut, radar): `<!-- chart-plot-area: <type> | center: cx,cy | radius: r -->`
- Located inside `<g id="chartArea">`, after axes, before data elements

## Executor Rules (from SKILL.md)

- **DO NOT redraw** a chart from scratch if `page_charts` maps a page to a template key — adapt the existing SVG (colors, typography, data density)
- **Adaptation scope:** replace placeholder data/labels, recolor series via spec_lock theme, adjust font sizes for text overflow — preserve layout structure
- **Coordinate accuracy:** use SVGPositionValidator to map data values to pixel positions; expect 10-50px errors from manual calculation
- **Tables** (basic_table, consulting_table, etc.): reflow row/column content, preserve column widths and header styling
- **Non-chart visuals** (process_flow, timeline, hub_spoke, etc.): replace labels, keep arrow/connector geometry

## Anti-Patterns (hard failure)

| Anti-pattern | Reason |
|---|---|
| `rgba()` / `<g opacity>` / `<style>` in chart SVGs | PPT renderer silently drops them |
| Copying template SVG verbatim without adapting colors | Spec_lock theme mismatch |
| Redrawing chart from scratch when `page_charts` entry exists | Violates SKILL.md executor rules |
| Omitting coordinate markers on calculator-supported charts | verify-charts workflow needs them |
| Chinese placeholder text in templates | CHART_STYLE_GUIDE §8.0: English-only placeholders |
| Registering new chart in index without `summary` field | Strategist selection needs the rule |
| Writing `summary` as description instead of selection rule | Format: "Pick for X. Skip if Y (use alternative)." |
