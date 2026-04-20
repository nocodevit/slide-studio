# Table Patterns for Slides

Tables inside presentations must obey the **100vh, no-scroll** rule. That constraint dictates every pattern below. If your data doesn't fit these shapes, it belongs in a dashboard or a handout, not a slide.

---

## When to Use a Table (vs Bullets / Cards / Chart)

**Tables are never the default.** They're chosen only when the content earns one. Use this decision flow **before** generating the slide.

### Explicit triggers (go straight to table, still confirm with user)

User intent is unambiguous when any of these is true:

- User literally says "table", "matrix", "comparison", "对比表", "tier list"
- User provides data in CSV / TSV / pipe-delimited form
- User shares a screenshot/text of an existing table and wants it on a slide
- Content is a **pricing tier page**, **feature matrix**, **spec sheet**, or **SLA breakdown**

### Heuristic signals (propose a table, let user choose)

Count these signals in the content. **2 or more ⇒ propose a table.**

1. **≥ 3 entities** (products, plans, teams, quarters) each with **≥ 2 shared attributes**
2. Bullets share a repeated shape: `X: A, Y: B, Z: C` — structured rows disguised as prose
3. The user's goal is explicitly **compare / pick / decide** (not describe or narrate)
4. Numbers dominate and need to be read across rows (ranking, benchmarks)
5. User used words: compare, vs, versus, 对比, 差异, benchmark, trade-off, pros/cons

### Reject a table (use something else) when any is true

- **Rows ≥ 9 or columns ≥ 7** after trimming → too dense for 100vh; switch to chart or split slides
- **Only 1 attribute per entity** → use a ranking list or bar chart, not a table
- Content is **narrative, emotional, or sequential** → bullets or timeline
- Entities are **unrelated** (no shared attribute axis) → cards/feature grid
- Audience goal is **one takeaway** → big-number slide or winner callout, not a whole grid

### Confirmation rule (A + B)

**Always** ask the user via `AskUserQuestion` before committing to a table — even on explicit triggers. Offer the table + 1–2 alternatives:

> Header: "Format"
> Question: "This content has 4 plans × 5 attributes — I'd render it as a **winner-highlighted comparison table** (Pattern 5). Alternatives: a **feature grid** of 4 cards, or a **ranking list**. Which do you prefer?"
> Options: Winner table (recommended) / Feature grid / Ranking list / Let me describe

This keeps the AI's judgment transparent and gives the user veto power without forcing them to describe formats they don't know.

---

## Hard Constraints (non-negotiable)

- **Rows (including header):** ≤ 8
- **Columns:** ≤ 6
- **No `overflow: auto` or `overflow: scroll`** on the table or any ancestor
- **Font size:** `clamp(0.8rem, 1.6vh, 1.15rem)`
- **Row height:** `clamp(2.2rem, 5vh, 3rem)`
- **No horizontal scroll** — if columns don't fit, drop columns or switch pattern
- **Interactivity:** CSS-only or native `<details>`. No JS frameworks, no external libraries. One small vanilla JS listener is fine (≤ 15 lines).
- **Expansion / drawer states must still fit 100vh** when fully open — test by opening every expandable row or drawer at once

**Exceeds limits? Split into multiple slides, switch to a visualization, or move it off-slide.**

---

## Pattern 1: Static Comparison Table

For 3–5 columns × 4–6 rows. The default shape. No interaction.

**When:** feature matrices, pricing tiers, before/after, competitor comparisons.

**Design moves:**

- Strong first-column emphasis (bold, slightly larger, colored)
- Header row visually distinct — uppercase letter-spacing, accent background, or bottom border only
- Thin horizontal dividers, **no vertical grid lines** (they read as cluttered on slides)
- Generous row padding (`clamp(0.8rem, 2vh, 1.4rem)` vertical)
- Monospace digits (`font-variant-numeric: tabular-nums`) for number columns

```html
<table class="slide-table">
  <thead>
    <tr><th>Feature</th><th>Basic</th><th>Pro</th><th>Enterprise</th></tr>
  </thead>
  <tbody>
    <tr><td class="row-label">Users</td><td>5</td><td>50</td><td>Unlimited</td></tr>
    <!-- max 5 more rows -->
  </tbody>
</table>
```

---

## Pattern 2: Expand-on-Click Detail Rows

Each row reveals extra context via native `<details>`. The clicked row grows downward with a CSS transition.

**When:** teaching, documentation walkthroughs, agenda slides with speaker notes visible on demand.

**Constraints:**

- **Max 5 base rows** (so one expansion still fits 100vh)
- Expanded content: ≤ 2 short paragraphs OR 3–4 bullets
- Only **one** row expanded at a time (use `name` attribute on `<details>` — exclusive group)
- Animate `max-height`, not `height` (auto isn't transitionable)

```html
<div class="expand-table">
  <div class="expand-header">
    <span>Phase</span><span>Owner</span><span>Status</span>
  </div>
  <details name="phase-detail" class="expand-row">
    <summary>
      <span>Discovery</span><span>Design</span>
      <span class="tag tag-success">Done</span>
    </summary>
    <div class="expand-body">
      User interviews, competitive audit, and jobs-to-be-done framing
      completed 2026-03-10.
    </div>
  </details>
  <!-- max 4 more -->
</div>
```

Key CSS:

```css
.expand-body {
  max-height: 0;
  overflow: hidden;
  transition: max-height 0.35s ease, padding 0.35s ease;
}
details[open] .expand-body {
  max-height: 20vh;
  padding-block: clamp(0.6rem, 1.5vh, 1rem);
}
```

---

## Pattern 3: Dropdown-Filtered Table

A single `<select>` changes which rows are visible. One vanilla JS listener toggles `hidden`.

**When:** segmenting the same metric by region/team/quarter, showing one view at a time.

**Constraints:**

- Total visible rows per filter value ≤ 7; total DOM rows ≤ 20 (keep the slide file small)
- Filter control sits above the table, ≤ 1 line tall
- Include an "All" option **only** if the full list still fits in 100vh

```html
<label class="filter">
  View by
  <select id="quarter-filter">
    <option value="q1">Q1</option>
    <option value="q2">Q2</option>
    <option value="q3">Q3</option>
  </select>
</label>
<table class="slide-table">
  <tr data-q="q1"><td>…</td></tr>
  <tr data-q="q2" hidden><td>…</td></tr>
</table>
<script>
  document.getElementById('quarter-filter').addEventListener('change', e => {
    document.querySelectorAll('[data-q]').forEach(r => {
      r.hidden = r.dataset.q !== e.target.value;
    });
  });
</script>
```

---

## Pattern 4: Inline Data Bars

Replace raw numbers with horizontal bars inside cells. Feels like a table, reads like a dashboard.

**When:** showing magnitudes, progress, distributions where relative scale matters more than exact value.

**Constraints:**

- Bar width = percentage of the max value in that column
- Keep the numeric label next to the bar (right-aligned) — bars alone lose precision
- **Single accent color**, not rainbow — vary length, not hue

```html
<td class="bar-cell">
  <span class="bar" style="--w: 78%"></span>
  <span class="num">78%</span>
</td>
```

```css
.bar-cell { position: relative; }
.bar {
  display: inline-block;
  width: var(--w);
  height: clamp(0.5rem, 1.2vh, 0.8rem);
  background: var(--accent);
  border-radius: 2px;
}
```

---

## Pattern 5: Winner-Highlighted Comparison

Every row has a clear "winner" cell marked with color, icon, or badge. Removes cognitive load from the audience.

**When:** any comparison where you want the audience to reach a conclusion fast (pitch decks, proposals).

**Constraints:**

- Exactly one winner per row (or per column — pick one axis and stay consistent)
- Winner cell: filled background + stronger weight, or a small badge ("✓ Best", "Winner")
- Losing cells stay legible — dim to **70% opacity max**, never invisible

```html
<tr>
  <td class="row-label">Onboarding time</td>
  <td>2 weeks</td>
  <td class="winner">3 days <span class="badge">Best</span></td>
  <td>1 week</td>
</tr>
```

---

## Pattern 6: Push-Aside Detail Drawer (NOT Overlay)

Click a row → a detail panel slides in from the right. The table **compresses to the left**; the drawer takes the right column. It is **not a modal** and **does not overlay** the table — both remain visible and legible, side by side.

**When:** dense tables where the audience wants to drill into one row without losing context. Great for case studies, customer records, ticket triage walk-throughs, student profiles.

### Layout model

Use CSS Grid with an animatable `grid-template-columns`. Default: one column (table full width). When a row is active: two columns (table ~57%, drawer ~43%).

**Key rules:**

- **Max 5 base rows** so the compressed table still fits without squishing
- **Drawer content ≤ 20 lines** of mixed text/bullets/tags — must fit 100vh
- Drawer closes on: click same row again, click outside, or press Esc
- Only **one row active** at a time (drawer never stacks)
- Compressed table must NOT get horizontal scroll — drop columns or shrink typography proportionally
- Respect `prefers-reduced-motion`: skip the transition, just toggle

### HTML

```html
<div class="drawer-layout" data-drawer="closed">
  <table class="slide-table drawer-table">
    <thead>…</thead>
    <tbody>
      <tr data-row="acme">
        <td>Acme Corp</td>
        <td><span class="tag tag-warning">At risk</span></td>
        <td class="bar-cell"><span class="bar" style="--w:42%"></span><span class="num">42%</span></td>
      </tr>
      <!-- max 4 more -->
    </tbody>
  </table>

  <aside class="drawer" aria-hidden="true">
    <header class="drawer-head">
      <h3 class="drawer-title"></h3>
      <button class="drawer-close" aria-label="Close">×</button>
    </header>
    <div class="drawer-body"></div>
  </aside>
</div>

<!-- Row detail content, kept out of the table, referenced by data-row -->
<template id="detail-acme">
  <p>Q3 revenue down 18% after their sponsor left. NPS still healthy at 62.</p>
  <ul>
    <li>Last meeting: 2026-03-22</li>
    <li>Open tickets: 4 (2 high)</li>
  </ul>
  <span class="tag tag-danger">Renewal in 30 days</span>
</template>
```

### CSS (animatable grid columns)

```css
.drawer-layout {
  display: grid;
  grid-template-columns: 1fr 0fr;
  gap: 0;
  transition: grid-template-columns 0.4s cubic-bezier(0.4, 0, 0.2, 1);
  height: 100%;
}
.drawer-layout[data-drawer="open"] {
  grid-template-columns: 1.3fr 1fr; /* table ~57%, drawer ~43% */
  gap: clamp(1rem, 2.5vw, 2rem);
}

.drawer {
  overflow: hidden; /* critical: no scroll even here */
  opacity: 0;
  transform: translateX(8px);
  transition: opacity 0.3s 0.1s, transform 0.3s 0.1s;
  border-left: 1px solid var(--muted, #e5e5e5);
  padding-inline-start: clamp(0.8rem, 2vw, 1.6rem);
}
.drawer-layout[data-drawer="open"] .drawer {
  opacity: 1;
  transform: translateX(0);
}

.drawer-table tbody tr {
  cursor: pointer;
  transition: background 0.2s;
}
.drawer-table tbody tr[data-active] {
  background: var(--accent-soft, rgba(0, 0, 0, 0.05));
  box-shadow: inset 3px 0 0 var(--accent);
}

@media (prefers-reduced-motion: reduce) {
  .drawer-layout, .drawer { transition: none; }
}
```

### JS (vanilla, ~15 lines)

```js
const layout = document.querySelector('.drawer-layout');
const drawer = layout.querySelector('.drawer');
const title = drawer.querySelector('.drawer-title');
const body = drawer.querySelector('.drawer-body');

layout.querySelectorAll('tbody tr[data-row]').forEach(row => {
  row.addEventListener('click', () => {
    const id = row.dataset.row;
    const active = layout.querySelector('tr[data-active]');
    if (active === row) { close(); return; }
    active?.removeAttribute('data-active');
    row.setAttribute('data-active', '');
    title.textContent = row.firstElementChild.textContent;
    body.replaceChildren(document.getElementById(`detail-${id}`).content.cloneNode(true));
    layout.dataset.drawer = 'open';
    drawer.setAttribute('aria-hidden', 'false');
  });
});
function close() {
  layout.dataset.drawer = 'closed';
  drawer.setAttribute('aria-hidden', 'true');
  layout.querySelector('tr[data-active]')?.removeAttribute('data-active');
}
drawer.querySelector('.drawer-close').addEventListener('click', close);
document.addEventListener('keydown', e => e.key === 'Escape' && close());
```

### When to choose drawer vs expand-row (Pattern 2)

| Use drawer (Pattern 6)                      | Use expand-row (Pattern 2)              |
| ------------------------------------------- | --------------------------------------- |
| Detail is 5–20 lines / rich content         | Detail is 1–3 lines / one paragraph     |
| Detail includes tags, bars, sub-lists       | Plain prose or short bullet list        |
| Want row highlight to persist while reading | Sequential reveal, one open at a time   |
| Table has ≤ 5 rows                          | Table has ≤ 5 rows (same limit)         |

---

## Cell Style Library

Reusable cell decorations. Include the CSS in the slide's `<style>` block and apply classes as needed.

### Tag / Pill

Compact inline label. Use for categories, states, priorities.

```html
<span class="tag">Draft</span>
<span class="tag tag-success">Shipped</span>
<span class="tag tag-warning">At risk</span>
<span class="tag tag-danger">Blocked</span>
<span class="tag tag-info">In review</span>
```

```css
.tag {
  display: inline-block;
  padding: clamp(0.2rem, 0.5vh, 0.35rem) clamp(0.5rem, 1vw, 0.8rem);
  border-radius: 999px;
  font-size: clamp(0.7rem, 1.3vh, 0.9rem);
  font-weight: 600;
  letter-spacing: 0.02em;
  line-height: 1.2;
  background: var(--muted, #eee);
  color: var(--fg, #222);
}
.tag-success { background: color-mix(in oklab, #16a34a 15%, transparent); color: #166534; }
.tag-warning { background: color-mix(in oklab, #f59e0b 18%, transparent); color: #92400e; }
.tag-danger  { background: color-mix(in oklab, #dc2626 15%, transparent); color: #991b1b; }
.tag-info    { background: color-mix(in oklab, #2563eb 13%, transparent); color: #1e40af; }
```

Tune these hex values to match the slide's aesthetic — never ship the defaults as-is on a non-neutral theme.

### Status Dot

Minimal pre-label indicator. Pair with text for screen readers.

```html
<td><span class="dot dot-ok"></span> Online</td>
<td><span class="dot dot-warn"></span> Degraded</td>
<td><span class="dot dot-down"></span> Down</td>
```

```css
.dot {
  display: inline-block;
  width: clamp(0.5rem, 1.2vh, 0.75rem);
  aspect-ratio: 1;
  border-radius: 50%;
  margin-inline-end: 0.4em;
  vertical-align: middle;
}
.dot-ok   { background: #16a34a; box-shadow: 0 0 0 3px rgba(22, 163, 74, 0.15); }
.dot-warn { background: #f59e0b; box-shadow: 0 0 0 3px rgba(245, 158, 11, 0.15); }
.dot-down { background: #dc2626; box-shadow: 0 0 0 3px rgba(220, 38, 38, 0.15); }
```

### Progress / Score Bar

See Pattern 4 for the full block. Quick cell version:

```html
<td><div class="score"><span style="--s: 82%"></span></div></td>
```

```css
.score { height: 6px; background: var(--muted); border-radius: 3px; overflow: hidden; }
.score > span { display: block; width: var(--s); height: 100%; background: var(--accent); }
```

### Trend Arrow

For delta columns. Use `▲` / `▼` with color and a numeric delta.

```html
<td class="trend up">▲ 12%</td>
<td class="trend down">▼ 4%</td>
<td class="trend flat">—</td>
```

```css
.trend { font-variant-numeric: tabular-nums; font-weight: 600; }
.trend.up   { color: #16a34a; }
.trend.down { color: #dc2626; }
.trend.flat { color: var(--muted-fg, #888); }
```

### Avatar / Identity Cell

For people / team columns. Initials circle + name.

```html
<td class="who"><span class="avatar">MY</span> Mei Yang</td>
```

```css
.who { display: inline-flex; align-items: center; gap: clamp(0.4rem, 1vw, 0.7rem); }
.avatar {
  display: inline-grid; place-items: center;
  width: clamp(1.6rem, 3.5vh, 2.2rem); aspect-ratio: 1;
  border-radius: 50%;
  background: var(--accent); color: #fff;
  font-size: clamp(0.65rem, 1.2vh, 0.8rem); font-weight: 700;
  letter-spacing: 0.02em;
}
```

### Rating / Stars

For qualitative scores.

```html
<td class="rating" data-score="4"><span></span><span></span><span></span><span></span><span></span></td>
```

```css
.rating { display: inline-flex; gap: 2px; }
.rating span { width: clamp(0.55rem, 1.2vh, 0.75rem); aspect-ratio: 1; background: var(--muted); clip-path: polygon(50% 0,61% 35%,98% 35%,68% 57%,79% 91%,50% 70%,21% 91%,32% 57%,2% 35%,39% 35%); }
.rating[data-score="5"] span { background: #f59e0b; }
.rating[data-score="4"] span:nth-child(-n+4) { background: #f59e0b; }
.rating[data-score="3"] span:nth-child(-n+3) { background: #f59e0b; }
.rating[data-score="2"] span:nth-child(-n+2) { background: #f59e0b; }
.rating[data-score="1"] span:nth-child(-n+1) { background: #f59e0b; }
```

---

## Choosing the Pattern

| Audience goal                              | Pattern                                   |
| ------------------------------------------ | ----------------------------------------- |
| Compare a few things across features       | 1 (static) or 5 (winner)                  |
| Brief reveal of one extra line per row     | 2 (expand)                                |
| Show different cuts of one dataset         | 3 (dropdown)                              |
| Feel the shape of the numbers              | 4 (data bars) or cell-level score bar     |
| Drive a clear conclusion                   | 5 (winner)                                |
| Drill into rich detail without losing grid | **6 (push-aside drawer)**                 |

---

## Styling Notes (match the presentation aesthetic)

- Use the same CSS variables the presentation uses (`--bg`, `--fg`, `--accent`, `--muted`, `--accent-soft`)
- Table typography should match the slide's body font — don't default to system
- Header row: uppercase + letter-spacing works well with most aesthetics; tone down for Notebook / Paper / Vintage Editorial styles
- **Never** use default browser table borders — design them or omit them
- Add a subtle entrance animation on the table (fade + translateY 8px, staggered per row via `animation-delay: calc(var(--i) * 40ms)`)
- Tag colors must be re-tuned per theme — the defaults above are calibrated for neutral light themes; dark themes need lighter foregrounds and dimmer backgrounds

## Accessibility

- Real `<table>` + `<thead>` + `<tbody>` semantics — not divs styled as tables (Patterns 2 and 6 use semantic alternatives with proper ARIA: `role="table"`, `aria-expanded`, `aria-hidden`)
- `scope="col"` on every `<th>`
- `<details>` (Pattern 2) handles keyboard and screen reader announcements for free
- Filter `<select>` (Pattern 3) must have an associated `<label>`
- Drawer (Pattern 6): toggle `aria-hidden` on the `<aside>`, focus the close button on open, return focus to the triggering row on close, support Esc
- **Color is never the only signal** — winners need a badge/icon, status needs a text label next to the dot, data bars need numbers, trends need ▲/▼ glyphs as well as color
