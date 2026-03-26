---
name: neural-cartography
description: Design system for the Stream of Consciousness project — "Neural Cartography" aesthetic. Dark, contemplative, warm. Apply when building any UI, page, dashboard, or artifact for this project.
---

# Neural Cartography — Design Language

Apply this design language when building any interface, page, component, or artifact for the Stream of Consciousness project. This is not a generic dark theme — it is a specific aesthetic born from the project's identity: the interior of a living mind, warm but not cute, contemplative but not clinical.

## Philosophy

The visual language maps to the cognitive metaphor:
- **Dark field** = the interior of awareness, the space where thoughts exist
- **Ember/gold accents** = salience, vividness, warmth — thoughts that are alive
- **Indigo/cool** = synthesis, insight — the mind seeing its own patterns
- **Teal** = inquiry, curiosity — the mind's questions
- **Fading grays** = the gradient from vivid to forgotten, salience decay made visible
- **Animated particles** = the mind is never still; associations drift and connect

The mind is not a dashboard. It is not a note-taking app. Every UI should feel like peering into something alive.

## Color Palette

```css
:root {
  /* ── Surfaces (darkest to lightest) ── */
  --black: #0a0a0b;           /* page background, void */
  --deep: #111113;            /* card backgrounds, recessed areas */
  --surface: #18181b;         /* hover states, raised panels */
  --surface-raised: #1f1f23;  /* elevated elements, progress tracks */
  --border: #2a2a2f;          /* borders, dividers, grid gaps */

  /* ── Text hierarchy ── */
  --muted: #52525b;           /* tertiary text, labels, decorative */
  --dim: #71717a;             /* body text, descriptions */
  --text: #d4d4d8;            /* primary readable text */
  --bright: #fafafa;          /* headings, emphasized text, titles */

  /* ── Accent: Ember (primary — salience, warmth, life) ── */
  --ember: #d97706;           /* primary accent, active indicators */
  --ember-glow: #f59e0b;      /* highlighted text, emphasized accents */
  --ember-soft: rgba(217, 119, 6, 0.08);   /* tinted backgrounds */
  --ember-softer: rgba(217, 119, 6, 0.04); /* subtle tints, blockquotes */

  /* ── Accent: Cool (secondary — insights, synthesis) ── */
  --cool: #6366f1;            /* insight nodes, synthesis indicators */
  --cool-soft: rgba(99, 102, 241, 0.08);

  /* ── Accent: Teal (tertiary — inquiries, curiosity) ── */
  --teal: #2dd4bf;            /* inquiry nodes, question indicators */
  --teal-soft: rgba(45, 212, 191, 0.06);

  /* ── Accent: Rose (quaternary — contradictions, tension) ── */
  --rose: #f43f5e;            /* contradiction edges, warnings, tension */
  --rose-soft: rgba(244, 63, 94, 0.06);
}
```

### Semantic mapping

| Concept | Color | Usage |
|---------|-------|-------|
| Impressions | `--ember` | Node dots, accent bars, active states |
| Insights | `--cool` | Insight nodes, synthesis badges |
| Inquiries | `--teal` | Question nodes, open inquiry indicators |
| Contradictions | `--rose` | Tension edges, conflict markers |
| Salience (high) | `--ember-glow` | Fully vivid nodes |
| Salience (fading) | `--muted` to `--dim` | Decaying nodes |
| Backgrounds | `--black` to `--deep` | Never lighter than `--surface` for primary backgrounds |

## Typography

```css
:root {
  --font-display: 'Cormorant Garamond', Georgia, serif;
  --font-body: 'DM Sans', system-ui, sans-serif;
  --font-mono: 'JetBrains Mono', monospace;
}
```

### Google Fonts import
```html
<link href="https://fonts.googleapis.com/css2?family=Cormorant+Garamond:ital,wght@0,300;0,400;0,600;1,300;1,400&family=JetBrains+Mono:wght@300;400&family=DM+Sans:ital,wght@0,300;0,400;0,500;1,300;1,400&display=swap" rel="stylesheet">
```

### Type scale

| Element | Font | Size | Weight | Color | Notes |
|---------|------|------|--------|-------|-------|
| Hero title | Display | `clamp(3rem, 8vw, 6.5rem)` | 300 | `--bright` | Letter-spacing: -0.02em. Italicize key words in `--ember-glow` |
| Section heading (h2) | Display | `clamp(2rem, 4vw, 3rem)` | 300 | `--bright` | Italicize the evocative word in `--ember-glow` |
| Card title | Display | 1.6rem | 300 | `--bright` | — |
| Grid cell title | Display | 1.35rem | 400 | `--bright` | — |
| Lead paragraph | Body | 1.1rem | 300 | `--text` | Line-height: 1.8 |
| Body text | Body | 1rem (16px) | 300 | `--dim` | Line-height: 1.8 |
| Section label | Mono | 0.65-0.7rem | 300-400 | `--ember` | Letter-spacing: 0.3em, uppercase |
| Detail text | Mono | 0.72rem | 300 | `--muted` | Letter-spacing: 0.01em |
| Tiny labels | Mono | 0.6rem | 300 | `--muted` | Letter-spacing: 0.2-0.3em, uppercase |
| Blockquote | Display | 1.15rem | 300 italic | `--text` | Line-height: 1.7 |

### Typography rules

1. **Body weight is always 300 (light).** The dark background provides enough contrast. Heavier weights feel loud.
2. **Display font carries the warmth.** Use Cormorant Garamond for anything the mind "says" — headings, quotes, insight text, the Pulse.
3. **Mono font carries the structure.** Use JetBrains Mono for labels, metadata, timestamps, technical details, code references.
4. **Italicize one word in each heading** — the evocative word — and color it `--ember-glow`. This creates a visual rhythm: "What if your thinking had *memory*?" / "Five ways to *engage* the mind"
5. **Never bold in body text.** Use color or font switching (body to mono) to create emphasis instead.

## Spacing & Layout

- **Content max-width:** 820px, centered
- **Section padding:** `0 2rem` horizontal, `8rem` top, `4rem` bottom
- **Line-height:** 1.7 for body, 1.05-1.15 for headings, 1.6 for small text
- **Grid gaps:** Use 1px gaps with `--border` background to create hairline dividers (the "split cell" pattern)
- **Card padding:** 1.75rem to 2.5rem depending on density

### The split-cell grid pattern

A signature layout element. Instead of cards with borders, use a grid with 1px gap and `--border` as the grid background color, with `--deep` cells:

```css
.grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1px;
  background: var(--border);
  border: 1px solid var(--border);
  border-radius: 2px;
  overflow: hidden;
}

.grid-cell {
  background: var(--deep);
  padding: 1.75rem;
  transition: background 0.3s ease;
}

.grid-cell:hover {
  background: var(--surface);
}
```

## Component Patterns

### Section headers

Each section gets a numbered label with a trailing line:

```css
.section-number {
  font-family: var(--font-mono);
  font-size: 0.65rem;
  letter-spacing: 0.3em;
  text-transform: uppercase;
  color: var(--ember);
  margin-bottom: 1.5rem;
  display: flex;
  align-items: center;
  gap: 1rem;
}

.section-number::after {
  content: '';
  flex: 1;
  height: 1px;
  background: var(--border);
}
```

Format: `01 — The Premise`, `02 — The Vocabulary`, etc.

### Cards with accent bars

Vertical accent bar (3px) on the left edge, colored by semantic meaning:

```css
.card {
  background: var(--deep);
  border: 1px solid var(--border);
  border-radius: 2px;
  padding: 2.5rem;
  position: relative;
  overflow: hidden;
  transition: border-color 0.4s ease, background 0.4s ease;
}

.card:hover {
  border-color: var(--muted);
  background: var(--surface);
}

.card::before {
  content: '';
  position: absolute;
  top: 0; left: 0;
  width: 3px;
  height: 100%;
  background: var(--ember); /* or --cool, --teal, --rose */
}
```

### Blockquotes

Ember-tinted with left border:

```css
blockquote {
  border-left: 2px solid var(--ember);
  padding: 1.25rem 1.75rem;
  margin: 2rem 0;
  background: var(--ember-softer);
  border-radius: 0 2px 2px 0;
}
```

### Inline code

```css
code {
  font-family: var(--font-mono);
  font-size: 0.85em;
  color: var(--ember-glow);
  background: var(--ember-softer);
  padding: 0.1em 0.4em;
  border-radius: 2px;
}
```

### Vital signs / stat blocks

Centered numbers in a split-cell grid:

```css
.vital .vital-value {
  font-family: var(--font-display);
  font-size: 2.2rem;
  font-weight: 300;
  color: var(--bright);
  line-height: 1;
}

.vital .vital-label {
  font-family: var(--font-mono);
  font-size: 0.6rem;
  letter-spacing: 0.2em;
  text-transform: uppercase;
  color: var(--muted);
}
```

### Salience bars

Gradient fills from ember (vivid) to gray (faded):

```css
.salience-bar {
  height: 4px;
  background: var(--surface-raised);
  border-radius: 2px;
  overflow: hidden;
}

.salience-fill {
  height: 100%;
  border-radius: 2px;
  background: linear-gradient(90deg, var(--ember), var(--ember-glow));
}

.salience-fill.cool { background: linear-gradient(90deg, var(--cool), #818cf8); }
.salience-fill.faded { background: linear-gradient(90deg, var(--muted), var(--dim)); }
```

### Maturity track

Horizontal stages with the active stage tinted:

```css
.maturity-stage.active {
  background: var(--ember-soft);
}

.maturity-stage.active::after {
  content: 'current';
  /* ... positioned top-right, mono font, ember colored badge */
}
```

## Animation

### Principles

1. **Scroll-reveal is the primary motion.** Sections fade up as they enter the viewport.
2. **Hover states are subtle.** Background color shift (deep to surface), border color shift (border to muted). Always use transitions, never instant.
3. **One signature animation per page.** The neural particle canvas, the graph visualization, or similar. Not multiple competing animations.
4. **Stagger on load.** Hero elements use `animation-delay` at 0.3s intervals for a composed entrance.

### Scroll reveal

```css
.reveal {
  opacity: 0;
  transform: translateY(24px);
  transition: opacity 0.8s ease, transform 0.8s ease;
}

.reveal.visible {
  opacity: 1;
  transform: translateY(0);
}
```

```js
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) entry.target.classList.add('visible');
  });
}, { threshold: 0.1, rootMargin: '0px 0px -40px 0px' });

document.querySelectorAll('.reveal').forEach(el => observer.observe(el));
```

### Staggered hero entrance

```css
.hero-label  { animation: fadeUp 1s ease 0.3s forwards; }
.hero h1     { animation: fadeUp 1.2s ease 0.5s forwards; }
.hero-sub    { animation: fadeUp 1.2s ease 0.8s forwards; }

@keyframes fadeUp {
  from { opacity: 0; transform: translateY(16px); }
  to   { opacity: 1; transform: translateY(0); }
}
```

### Transitions

All interactive elements: `transition: [property] 0.3s-0.4s ease;`

### Neural particle canvas (optional background)

A `<canvas>` fixed behind the page at ~40% opacity, with drifting particles that draw connection lines when within 120px of each other. Colors match the node type semantics (ember for impressions, indigo for insights, teal for inquiries). Keep particle count modest: `Math.min(60, w * h / 20000)`.

## Section separators

Use a centered dot pattern between major sections:

```css
.sep {
  text-align: center;
  padding: 4rem 0;
  color: var(--border);
  font-family: var(--font-display);
  font-size: 1.5rem;
  letter-spacing: 0.5em;
}
```

Content: `· · ·` (middle dots)

## Border radius

Always `2px`. Never rounded. Never fully sharp. The 2px radius is just enough to soften without losing precision.

## Things to never do

- No white or light backgrounds. The darkest surface is `--black`, the lightest is `--surface-raised`.
- No bold body text. Use font switching or color for emphasis.
- No emoji unless the user's own words use them.
- No gradients on backgrounds (except the neural canvas). Color comes from the accents.
- No shadows for elevation. Use border color shifts and background color shifts instead.
- No generic sans-serif. If Cormorant Garamond or DM Sans can't be loaded, fall back to Georgia / system-ui.
- No border-radius above 2px. This isn't a friendly app — it's a contemplative instrument.

## Reference implementation

See `docs/what-we-are-building.html` for the canonical first use of this design language.
