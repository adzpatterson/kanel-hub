# CLAUDE.md — Kanel Supply Chain Hub

## Project Overview

Kanel Supply Chain Hub is a single-file HTML application (`index.html`) serving as a supply chain management and inventory optimization dashboard for Kanel Spice Company. It integrates with Fishbowl ERP via a local proxy service (`proxy.js` on `localhost:3847`).

**Current version:** 4.1.0

## Architecture

### Single-File SPA

The entire application lives in `index.html` (~2,900 lines) containing:
- **CSS** (lines 8–160): Dark theme with CSS variables, embedded in `<style>` tag
- **HTML** (lines 161–560): Tab-based layout, tables, modals, setup screen
- **Reference Data** (lines 563–596): Hardcoded constants (exclusions, MOQs, outside demand, discontinued items)
- **JavaScript** (lines 597–2880): All application logic in `<script>` tag

No build system, no framework, no package manager. Pure vanilla HTML/CSS/JS.

### Data Flow

1. `proxy.js` (localhost:3847) queries Fishbowl ERP REST API
2. Frontend fetches `/api/woh` from proxy → receives full dataset as JSON
3. JavaScript computes derived metrics (WOH, ABC analysis, RM demand, forecasts)
4. DOM is updated via innerHTML rendering
5. User preferences stored in `localStorage` with `kh-` prefix

### External Dependencies (loaded via CDN/API)

- Google Fonts (DM Sans, JetBrains Mono)
- XLSX library (Excel export)
- Google Drive API (Sheets export)
- Google Calendar API (planned)

## Code Conventions

### State Management

Global state object `S` holds all application data:
```javascript
let S = {
  config: null,     // proxy URL config
  data: null,       // raw Fishbowl data
  forecast: {},     // forecast data (localStorage-persisted)
  excluded: {},     // globally excluded items
  wohExcluded: {},  // WOH-specific exclusions
  sortC: {},        // sort column per tab
  sortD: {},        // sort direction per tab
  _c: {},           // cached computed data
  // ... tab-specific filters and flags
};
```

### Function Naming Patterns

- `calc*()` — Compute derived data (e.g., `calcWOH()`, `calcRMD()`, `calcABC()`)
- `r*()` — Render/update a full tab (e.g., `rWOH()`, `rRMD()`, `rExec()`)
- `r*T()` — Render a specific table within a tab (e.g., `rWOHT()`, `rRMDT()`)
- `srt(tab, col)` — Sort a table by column
- `switchTab(t)` — Navigate between tabs
- `inc(n)` / `incWOH(n)` — Inclusion checks for filtering items

### Helper Functions

- `fmt(n, d=0)` — Format number with locale (en-CA)
- `fmtDate(s)` — Format date string
- `bdg(s)` — Generate status badge HTML (`oos`, `critical`, `atrisk`, `healthy`, `excess`)
- `toKg(q, u)` — Convert quantity to kilograms
- `isWt(u)` — Check if unit is weight-based
- `getCat(n)` — Categorize product by name (regex-based)
- `fixTZ(s)` — Fix timezone offset in date strings from Fishbowl

### CSS Theming

Dark theme using CSS custom properties:
- `--bg`, `--surface`, `--surface-2` — Background layers
- `--accent: #d4a853` — Gold accent (brand color)
- `--red`, `--orange`, `--blue`, `--green`, `--gray` — Status colors
- `--font` — DM Sans; `--mono` — JetBrains Mono

### Status Badge System

Items are classified by weeks-on-hand coverage:
- **OOS** (gray) — Out of stock
- **CRITICAL** (red) — Below minimum threshold
- **AT RISK** (orange) — Low but not critical
- **HEALTHY** (blue) — Adequate coverage
- **EXCESS** (green) — Over target

### LocalStorage Keys

All prefixed with `kh-`:
- `kh-cfg` — Proxy URL configuration
- `kh-fc` — Forecast data
- `kh-fm` — Forecast months
- `kh-ml` — MO list
- `kh-excl` — Global exclusions
- `kh-woh-excl` — WOH-specific exclusions
- `kh-cl-*` — Changelog acknowledgments

## Tab Structure

12 main tabs + WIP sections:
1. **Executive Summary** (`exec`) — KPI cards, ABC analysis overview
2. **Weeks on Hand** (`woh`) — FG inventory coverage with filters
3. **RM Demand** (`rmdemand`) — Multi-level BOM explosion to raw materials
4. **Kanel Production** (`casepack`) — Internal production planning
5. **Axia Production** (`axia`) — Contract manufacturer production
6. **Manufacturing Orders** (`mos`) — MO workflow with sub-tabs (Open, RM Required, Schedule, Closed)
7. **Purchasing** (`purchasing`) — Vendor management, cash flow
8. **Open POs** (`pos`) — Purchase order tracking
9. **BOMs** (`boms`) — Bill of Materials browser
10. **Manage Items** (`manage`) — Item exclusion management
11. **Updates** — Changelog
12. **How It Works** — User guide

WIP tabs: Safety Stock, Production Calendar, Forecast Hub

## Key Business Logic

### Reference Data (Hardcoded)

- `AUTO_EXCL[]` — Product names auto-excluded from calculations (demos, signage, etc.)
- `MOQ{}` — Minimum order quantities per product
- `OUTSIDE_DEMAND{}` — Manual demand additions (retail partners, seasonal SKUs)
- `DISCONTINUED[]` — Discontinued product list
- `WOH_EXCLUDE[]` — Items hidden from WOH report
- `KANEL_PROD_RE` — Regex identifying Kanel-produced items (case packs, gifts, etc.)
- `VENDOR_LEAD_TIMES{}` — Weeks lead time per vendor
- `SAFETY_FACTOR` — 1.5 weeks safety stock buffer

### Product Categories (regex-based in `getCat()`)

Sea Salt, Spice Blends, Cocktail Rimmers, Gifts, Case Packs, Refills, Bulk

### Locations

- **Main** — Internal production facility
- **Axia** — Contract manufacturer

## Development Workflow

### Deployment

1. Edit `index.html` directly
2. Replace file on deployment target
3. Restart proxy if `proxy.js` changed: `node proxy.js`
4. Refresh browser

### No Build/Test/Lint

There is no build step, test suite, or linter configured. Changes are validated manually in the browser.

### Git History

The repository uses a simple pattern of uploading `index.html` directly. Commits are typically "Add files via upload" / "Delete index.html" cycles from GitHub web UI uploads.

## Guidelines for AI Assistants

- **Single file**: All changes go in `index.html`. Do not split into multiple files unless explicitly requested.
- **No frameworks**: Keep it vanilla JS. Do not introduce React, Vue, or any framework.
- **Compact style**: The codebase uses a compact/minified style with minimal whitespace. Match this convention.
- **Naming**: Follow existing short function names and the `calc*`/`r*`/`r*T` pattern.
- **State**: Add new state to the global `S` object. Persist user preferences to `localStorage` with `kh-` prefix.
- **New tabs**: Follow the `switchTab()` pattern — add a `<div id="tab-name" class="tab-content hidden">` and a nav tab entry.
- **Tables**: Use the existing table/card/badge CSS classes. Render via innerHTML strings.
- **Sorting**: Register new sortable tables in the `srt()` function's render map.
- **Export**: Follow the existing XLSX export pattern for new tabs.
- **Reference data**: Hardcoded business data (MOQs, exclusions, lead times) lives at the top of the `<script>` block as constants.
