# SentinelAI — AI Threat Detection Dashboard
## Complete Technical Documentation

---

## Table of Contents
1. [Project Overview](#1-project-overview)
2. [Architecture](#2-architecture)
3. [File Structure](#3-file-structure)
4. [Design System](#4-design-system)
5. [Component Breakdown](#5-component-breakdown)
6. [Data Layer](#6-data-layer)
7. [JavaScript Functions Reference](#7-javascript-functions-reference)
8. [Repository Inspirations Used](#8-repository-inspirations-used)
9. [How to Run](#9-how-to-run)
10. [How to Connect a Real Backend](#10-how-to-connect-a-real-backend)
11. [Extending the Dashboard](#11-extending-the-dashboard)

---

## 1. Project Overview

SentinelAI is a **single-file, zero-dependency** AI-powered threat detection dashboard
built as pure HTML + CSS + Vanilla JavaScript.

It demonstrates:
- Real-time live threat monitoring UI
- AI/ML model performance tracking (XGBoost, Random Forest, LSTM, Isolation Forest, SVM)
- Geographic attack origin visualization
- Authentication failure analytics
- Interactive response actions

**Tech stack used:**
| Dependency      | Version | Purpose                          |
|-----------------|---------|----------------------------------|
| Chart.js        | 4.4.1   | Donut chart for threat types     |
| Tabler Icons    | 3.34.0  | 5800+ outline SVG icon font      |
| Google Fonts    | —       | DM Sans (UI) + JetBrains Mono (data) |
| Vanilla JS      | ES2020  | All interactivity, no frameworks |

---

## 2. Architecture

```
┌─────────────────────────────────────────────────────────┐
│                        index.html                        │
│                                                         │
│  ┌──────────────────────────────────────────────────┐   │
│  │  <head>  Design tokens (CSS vars) + CDN links    │   │
│  └──────────────────────────────────────────────────┘   │
│                                                         │
│  ┌──────────────────────────────────────────────────┐   │
│  │  <body class="app">                              │   │
│  │   ├── <header class="topbar">   (top nav)        │   │
│  │   ├── <nav class="sidebar">     (left nav)       │   │
│  │   └── <main class="main">       (content area)   │   │
│  │        ├── .metrics              4-col KPI row   │   │
│  │        ├── .mid                  threat + geo    │   │
│  │        ├── .bottom               models + alerts │   │
│  │        └── .card.actions         response btns   │   │
│  └──────────────────────────────────────────────────┘   │
│                                                         │
│  ┌──────────────────────────────────────────────────┐   │
│  │  <script>  DATA arrays → render functions        │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

### Layout mechanism

The root `.app` element uses **CSS Grid** with two named areas:

```css
.app {
  display: grid;
  grid-template-rows: 54px 1fr;    /* topbar height | rest */
  grid-template-columns: 220px 1fr; /* sidebar | main */
  height: 100vh;
  overflow: hidden;                 /* scrolling lives in .main only */
}
```

The topbar spans both columns (`grid-column: 1 / -1`).
The sidebar scrolls independently if it overflows.
The main area is the only scrollable region.

---

## 3. File Structure

```
sentinel-ai/
├── index.html          ← ENTIRE app lives here (self-contained)
├── README.md           ← This documentation file
│
└── src/                ← (optional) if you split into modules
    ├── components/
    │   ├── ThreatFeed.js
    │   ├── GeoOrigins.js
    │   ├── ModelPerf.js
    │   ├── AlertLog.js
    │   └── Metrics.js
    ├── hooks/
    │   └── usePolling.js    (if you add React)
    ├── data/
    │   └── mockData.js      (exported constants)
    └── utils/
        └── formatters.js    (number/time formatters)
```

For a single-file standalone demo: **only `index.html` is needed**.

---

## 4. Design System

### Color palette (CSS custom properties)

All colors are defined once in `:root` and referenced everywhere:

```css
:root {
  /* ── Backgrounds (dark theme, 3-level depth) */
  --bg-base:       #0a0c10;   /* page background */
  --bg-surface:    #111318;   /* cards, sidebar, topbar */
  --bg-elevated:   #181c24;   /* inner elements (rows, cells) */
  --bg-hover:      #1f2430;   /* hover states */

  /* ── Borders */
  --border:        rgba(255,255,255,.07);   /* subtle */
  --border-active: rgba(255,255,255,.16);   /* hover/focus */

  /* ── Text */
  --text-primary:  #e8ecf4;   /* main readable text */
  --text-secondary:#7c8496;   /* labels, sublabels */
  --text-tertiary: #4a5168;   /* timestamps, hints */

  /* ── Semantic colours (+ dim variants for backgrounds) */
  --red:        #e24b4a;   --red-dim:    rgba(226,75,74,.15);
  --amber:      #ef9f27;   --amber-dim:  rgba(239,159,39,.15);
  --blue:       #378add;   --blue-dim:   rgba(55,138,221,.15);
  --green:      #22c55e;   --green-dim:  rgba(34,197,94,.15);
  --purple:     #a78bfa;
}
```

### Typography

Two fonts:
- `DM Sans` — clean, modern, highly legible at small sizes. Used for all UI copy.
- `JetBrains Mono` — monospaced, precise. Used for numbers, IPs, timestamps, badges.

Why this pairing: data dashboards benefit from monospace numbers so digits align
and changing values feel "live". The contrast between a humanist sans and a
technical mono is deliberate.

### Severity colour mapping

| Severity | Foreground  | Background (dim)      | Use               |
|----------|-------------|----------------------|-------------------|
| critical | `--red`     | `--red-dim`          | Borders, pills    |
| high     | `--amber`   | `--amber-dim`        | Borders, pills    |
| medium   | `--blue`    | `--blue-dim`         | Borders, pills    |
| ok/pass  | `--green`   | `--green-dim`        | Status, accuracy  |

### Spacing scale

Used consistently throughout:
```
4px   — icon gap, tiny internal
6px   — badge padding, row gap
8px   — list gap
10px  — card padding internal
12px  — grid gap between cards
14px  — card padding
16px  — card padding (outer)
20px  — main content padding
```

---

## 5. Component Breakdown

### 5.1 Topbar

```html
<header class="topbar">
  <!-- Logo + live status pill -->
  <!-- Active alert badge counts -->
  <!-- Clock (updates every 1s via JS) -->
  <!-- Refresh button + Settings button -->
</header>
```

The `live-dot` inside the `.live-pill` pulses using a CSS animation:
```css
@keyframes pulse { 0%,100%{opacity:1} 50%{opacity:.3} }
```

### 5.2 Sidebar Navigation

Built as anchor tags with `.nav-item` + `.nav-section-label` dividers.
The `.active` class adds a red-tinted background and font-weight 500.

The notification badge `.nav-badge` is placed with `margin-left: auto` to push it
to the far right of the flex row.

### 5.3 Metric Cards

Each metric card has a **3px left border accent** using a `::before` pseudo-element.
The colour class (`danger`, `warn`, `info`, `ok`) on the parent sets the border colour:

```css
.metric::before {
  content: ''; position: absolute; left: 0; top: 0; bottom: 0;
  width: 3px; border-radius: 99px 0 0 99px;
}
.metric.danger::before { background: var(--red); }
```

This avoids `border-left` which would conflict with `border-radius` on the card.

### 5.4 Live Threat Feed

Rendered by `renderThreatFeed()`. Each row:
- Coloured circle icon (severity-matched background/foreground)
- Threat name (truncated with `text-overflow: ellipsis`)
- Source IP in monospace
- Severity pill badge
- Relative timestamp

Rows have `border: 1px solid transparent` by default, which becomes
`border-color: var(--border-active)` on hover — this prevents layout shift (a
common hover bug where adding a border shifts adjacent elements).

### 5.5 Geographic Origins Bar Chart

Built with CSS `flexbox` bars — no chart library needed.
The bar width is calculated as `(count / maxCount) * 100%` inline:

```js
const max = Math.max(...GEO_DATA.map(g => g.count));
// then per row:
style="width:${Math.round(g.count / max * 100)}%"
```

### 5.6 Model Performance Bars

Same technique as geo bars. Each bar's background colour is the model's semantic
colour, making it easy to compare at a glance.

The `<select>` filter calls `renderModels(value)` which slices the array to top 3.

### 5.7 Sparkline (24h Threat Volume)

A simple `flexbox` bar chart using `align-items: flex-end`. Each bar's height is:
```js
height: ${Math.round(v / max * 100)}%
```
The last bar has full opacity; all others are `0.45` — visually emphasising the
most recent data point.

### 5.8 Alert Log

A scrollable list (`max-height: 240px; overflow-y: auto`) of alert items.
Each has a `border-left: 3px solid <severity-color>` which creates the
coloured accent strip without affecting the flex/grid layout.

### 5.9 Donut Chart (Chart.js)

The only Chart.js usage. Key settings:
```js
{
  type: 'doughnut',
  options: {
    cutout: '68%',               // wide donut ring
    plugins: { legend: { display: false } }  // custom HTML legend instead
  }
}
```
Legend is built manually in HTML so it matches the design system's typography
and colours instead of Chart.js's defaults.

**Important**: canvas cannot read CSS variables, so hardcoded hex values are used:
`'#e24b4a', '#ef9f27', '#378add', '#a78bfa'`

### 5.10 Response Actions

Buttons use `.action-btn`. The primary (auto-contain) button has `.primary`
which adds a red-tinted background. All others are neutral.

In production, each `onclick` handler would call your backend API. Currently
they use `alert()` as stubs.

---

## 6. Data Layer

All data lives in `const` arrays at the top of the `<script>` block.
**In production, replace these with `fetch()` calls to your API.**

### THREATS array
```js
const THREATS = [
  {
    name: 'SQL injection burst',   // display name
    src:  '192.168.4.21',          // source IP / hostname
    sev:  'critical',              // 'critical' | 'high' | 'medium'
    icon: 'ti-code',               // Tabler icon class (without 'ti' prefix)
    ago:  '0:12s',                 // relative time string
  },
  // ...
];
```

### GEO_DATA array
```js
const GEO_DATA = [
  { flag: '🇨🇳', name: 'China', count: 312 },
  // count is absolute attack count (used to calc bar %)
];
```

### MODELS array
```js
const MODELS = [
  { name: 'XGBoost', acc: 98.7, color: '#22c55e' },
  // acc is a float (0–100), color is hex for the bar
];
```

### SPARK_DATA
```js
const SPARK_DATA = [34, 52, 28, ...]; // 24 numbers, one per hour
```

### ALERTS
```js
const ALERTS = [
  { sev: 'critical', text: 'Alert description text', ts: '2 min ago' },
];
```

---

## 7. JavaScript Functions Reference

| Function            | Purpose                                                 |
|---------------------|---------------------------------------------------------|
| `renderThreatFeed()`| Builds threat feed HTML from `THREATS` array            |
| `renderGeo()`       | Builds geographic bars from `GEO_DATA`                  |
| `renderModels(f)`   | Builds model bars; `f='all'` or `f='top'`               |
| `renderSparkline()` | Builds 24h sparkline from `SPARK_DATA`                  |
| `renderAlerts()`    | Builds alert log from `ALERTS`                          |
| `updateClock()`     | Updates `#clock` with `new Date().toLocaleTimeString()` |
| `refreshMetrics()`  | Randomises KPI metric values (simulates live poll)      |
| `initDonut()`       | Initialises Chart.js donut chart                        |
| `triggerContainment()` | Action stub — wire to backend                        |
| `exportReport()`    | Action stub — wire to backend                           |
| `runScan()`         | Action stub — wire to backend                           |
| `blockIPs()`        | Action stub — wire to backend                           |
| `notifyTeam()`      | Action stub — wire to backend                           |
| `viewModels()`      | Action stub — wire to backend                           |

---

## 8. Repository Inspirations Used

### posthog/posthog → analytics dashboard inspiration
- Multi-metric KPI cards with up/down trend deltas
- Left sidebar navigation with active state and badge counts
- Consistent dark surface depth (base → surface → elevated)

### tabler/tabler → admin dashboard design
- CSS Grid `grid-template-rows: topbar | content` layout pattern
- `.nav-section-label` grouping in sidebar
- Semantic badge colours (danger/warn/info/ok) as reusable classes

### ToolJet/ToolJet → enterprise dashboard UI
- Three-level content hierarchy (metrics → mid → bottom)
- Response action bar at the bottom for bulk operations
- Monospace font for technical data (IPs, timestamps, numbers)

### recharts/recharts → charts
- Pattern applied: Chart.js was used as the implementation, but Recharts'
  design philosophy influenced layout — custom HTML legends, responsive
  containers, data-first rendering via `data.map()` into DOM

### alan2207/bulletproof-react → production React structure
- Data arrays separated from render functions (clean separation of concerns)
- Explicit typed data shapes (the constants mimic typed interfaces)
- One render function per component

### fastapi/full-stack-fastapi-template → backend architecture
- Action stubs are structured to be replaced with `fetch('/api/...')` calls
- Data constants match REST response shapes (ready to swap in real API)

### nextauthjs/next-auth → authentication flow
- Auth metrics section (login count, failures, MFA coverage)
- Session/brute-force tracking concept reflected in `ALERTS` data

---

## 9. How to Run

### Option A — Just open in browser
```bash
open index.html
# or drag index.html into any browser tab
```
No build step. No server. No npm. Works offline.

### Option B — Local dev server (avoids CORS for future API calls)
```bash
# Python 3
python3 -m http.server 3000
# then open http://localhost:3000

# Node (if you have npx)
npx serve .
```

### Option C — Netlify / Vercel deploy (1-minute)
```bash
# Netlify drop
# Go to app.netlify.com/drop → drag the folder → done

# Vercel CLI
npm i -g vercel
vercel
```

---

## 10. How to Connect a Real Backend

### Step 1 — Replace data constants with API fetch

```js
// Replace this:
const THREATS = [ { name: 'SQL injection...', ... } ];

// With this:
async function loadThreats() {
  const res = await fetch('/api/v1/threats?limit=8');
  const data = await res.json();
  return data.threats; // adjust to your API shape
}
```

### Step 2 — Wire render functions to async data

```js
async function init() {
  const threats = await loadThreats();
  renderThreatFeed(threats);  // pass data as argument

  const geo = await loadGeoData();
  renderGeo(geo);

  // etc.
}
init();
```

### Step 3 — Add WebSocket for truly live updates

```js
const ws = new WebSocket('wss://your-api.com/ws/threats');
ws.onmessage = (event) => {
  const threat = JSON.parse(event.data);
  THREATS.unshift(threat);      // prepend to array
  THREATS.splice(8);            // keep only 8 newest
  renderThreatFeed();           // re-render
};
```

### Step 4 — FastAPI backend endpoint example

```python
# main.py (FastAPI)
from fastapi import FastAPI
app = FastAPI()

@app.get("/api/v1/threats")
def get_threats(limit: int = 8):
    # query your ML detection pipeline / SIEM
    return {"threats": threat_service.get_recent(limit)}

@app.get("/api/v1/models")
def get_model_performance():
    return {"models": model_registry.get_accuracy_stats()}
```

### Step 5 — Add auth (NextAuth pattern)

```js
// In your SPA init:
const session = await fetch('/api/auth/session').then(r => r.json());
if (!session.user) {
  window.location.href = '/login';
}
```

---

## 11. Extending the Dashboard

### Add a new KPI metric card

1. Add a new `.metric` div in the `.metrics` grid
2. Give it a class: `danger`, `warn`, `info`, or `ok`
3. Set `id` attributes on the value and sub elements
4. Update `refreshMetrics()` to include the new metric

### Add a new threat type to the donut

```js
// In initDonut():
data: [38, 27, 19, 16, 0],           // add new value
labels: ['Malware','Phishing','DDoS','Injection','NewType'],
backgroundColor: [...existing, '#NEW_COLOR'],
```

Also add a legend entry in the custom HTML legend below the chart.

### Add a new sidebar section

```html
<span class="nav-section-label">Your Section</span>
<a class="nav-item"><i class="ti ti-ICON"></i> Page Name</a>
```

Browse all 5800+ Tabler icons at: https://tabler.io/icons

### Add real-time polling (no WebSocket)

```js
// Add after init calls at bottom of script:
setInterval(async () => {
  const fresh = await loadThreats();
  renderThreatFeed(fresh);
}, 15_000); // poll every 15 seconds
```

---

## Key Design Decisions

| Decision                         | Rationale                                      |
|----------------------------------|------------------------------------------------|
| Single HTML file                 | Zero setup, easy to demo, deploy anywhere      |
| CSS variables for all colours    | Swap entire theme by editing `:root` only      |
| Monospace for numeric data       | Digits align; changing values feel precise     |
| `border: 1px solid transparent` on rows | No layout shift on hover border add    |
| `::before` pseudo-element for left accent | Avoids border-radius conflict      |
| Custom Chart.js legend           | Matches design system typography/colours       |
| `grid-template-columns: 220px 1fr` | Fixed sidebar, fluid content area            |
| `overflow: hidden` on `.app`     | Single scroll region in `.main` only           |

---

*Built for the SentinelAI threat detection system.*
*Stack: HTML5 · CSS3 (Grid/Flexbox/Custom Properties) · ES2020 JS · Chart.js 4 · Tabler Icons*
