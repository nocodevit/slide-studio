# Table Patterns for Slides

Tables inside presentations must still obey the **100vh, no-scroll** rule. That constraint dictates every pattern below. If your data doesn't fit these shapes, it belongs in a dashboard or a handout, not a slide.

## Hard Constraints (non-negotiable)

- **Rows (including header):** ≤ 8
- **Columns:** ≤ 6
- **No `overflow: auto` or `overflow: scroll`** on the table or any ancestor
- **Font size:** `clamp(0.8rem, 1.6vh, 1.15rem)`
- **Row height:** `clamp(2.2rem, 5vh, 3rem)`
- **No horizontal scroll** — if columns don't fit, drop columns or use Pattern 5 (grouped)
- **Interactivity:** CSS-only or native `<details>`. No JS frameworks, no external libraries.
- **Expansion states must still fit 100vh** when fully open — test by opening every expandable row at once

**Exceeds limits? Split into multiple slides, switch to a visualization (bar chart, ranking list), or move it off-slide.**

---

## Pattern 1: Static Comparison Table

For 3–5 columns × 4–6 rows. The default. No interaction.

**When to use:** feature matrices, pricing tiers, before/after, competitor comparisons.

**Design moves:**

- Strong first-column emphasis (bold, slightly larger, colored)
- Header row visually distinct — uppercase letter-spacing, accent background, or bottom border only
- Thin horizontal dividers, no vertical grid lines (vertical lines read as cluttered on slides)
- Generous row padding (`clamp(0.8rem, 2vh, 1.4rem)` vertical)
- Monospace for numbers — tabular alignment matters

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

**When to use:** teaching, documentation walkthroughs, agenda slides with speaker notes visible on demand.

**Constraints:**

- **Max 5 base rows** (so one expansion still fits 100vh)
- Expanded content: ≤ 2 short paragraphs OR 3–4 bullets
- Only ONE row expanded at a time (use `name` attribute on `<details>` — exclusive group)
- Animate `max-height`, not `height` (auto isn't transitionable)

```html
<div class="expand-table">
  <div class="expand-header">
    <span>Phase</span><span>Owner</span><span>Status</span>
  </div>
  <details name="phase-detail" class="expand-row">
    <summary>
      <span>Discovery</span><span>Design</span><span>Done</span>
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
  max-height: 20vh; /* cap — must leave room inside 100vh */
  padding-block: clamp(0.6rem, 1.5vh, 1rem);
}
```

---

## Pattern 3: Dropdown-Filtered Table

A single `<select>` changes which rows are visible. No JS framework — one `change` listener toggles `hidden` attribute or a `data-*` selector.

**When to use:** segmenting the same metric by region/team/quarter, showing one view at a time.

**Constraints:**

- Total rows across all filters ≤ 8 (so DOM stays small and no filter ever pushes past 100vh)
- Filter control sits above the table, ≤ 1 line tall
- Include an "All" option only if the full list fits

```html
<label class="filter">
  View by
  <select id="quarter-filter">
    <option value="q1">Q1</option>
    <option value="q2">Q2</option>
    <option value="q3">Q3</option>
  </select>
</label>
<table class="slide-table" data-filter="q1">
  <tr data-q="q1"><td>…</td></tr>
  <tr data-q="q2" hidden><td>…</td></tr>
</table>
<script>
  const sel = document.getElementById('quarter-filter');
  sel.addEventListener('change', e => {
    document.querySelectorAll('[data-q]').forEach(r => {
      r.hidden = r.dataset.q !== e.target.value;
    });
  });
</script>
```

---

## Pattern 4: Inline Data Bars

Replace raw numbers with horizontal bars inside cells. Feels like a table, reads like a dashboard.

**When to use:** showing magnitudes, progress, distributions where relative scale matters more than exact value.

**Constraints:**

- Bar width = percentage of max value in that column
- Keep the numeric label next to the bar (right-aligned) — bars alone lose precision
- Single accent color, not rainbow — varying only length, not hue

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

Every row has a clear "winner" cell marked with color, icon, or badge. Takes cognitive load off the audience.

**When to use:** any comparison where you want the audience to reach a conclusion fast (pitch decks, proposals).

**Constraints:**

- Exactly one winner per row (or per column, pick one axis)
- Winner cell: filled background + stronger weight, OR a small badge ("✓ Best", "Winner", custom mark)
- Losing cells stay legible — dim to 70% opacity max, never invisible

```html
<tr>
  <td class="row-label">Onboarding time</td>
  <td>2 weeks</td>
  <td class="winner">3 days <span class="badge">Best</span></td>
  <td>1 week</td>
</tr>
```

---

## Choosing the Pattern

| Audience goal                        | Pattern                  |
| ------------------------------------ | ------------------------ |
| Compare a few things across features | 1 (static) or 5 (winner) |
| Reveal detail without cluttering     | 2 (expand)               |
| Show different cuts of one dataset   | 3 (dropdown)             |
| Feel the shape of the numbers        | 4 (data bars)            |
| Drive a clear conclusion             | 5 (winner)               |

---

## Styling Notes (match the presentation aesthetic)

- Use the same CSS variables the presentation uses (`--bg`, `--fg`, `--accent`, `--muted`)
- Table typography should match the slide's body font, not default to system
- Header row: uppercase + letter-spacing works well with most aesthetics; tone down for Notebook/Paper styles
- Never use default browser table borders — design them or omit them
- Add a subtle entrance animation on the table (fade + translateY 8px, staggered per row via `animation-delay: calc(var(--i) * 40ms)`)

## Accessibility

- Real `<table>` + `<thead>` + `<tbody>` semantics — not divs styled as tables (Pattern 2 is the exception, but add ARIA roles)
- `scope="col"` on `<th>` elements
- `<details>` handles keyboard and screen reader announcements for free
- Filter `<select>` must have an associated `<label>`
- Color is never the only signal — winners need a badge/icon too, data bars need numbers
