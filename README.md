# CSS Concepts

## Table of Contents

1. [Box Model](#box-model)
2. [Selectors and Specificity](#selectors-and-specificity)
3. [Positioning](#positioning)
4. [Display and Layout](#display-and-layout)
5. [Flexbox](#flexbox)
6. [CSS Grid](#css-grid)
7. [Responsive Design](#responsive-design)
8. [Units](#units)
9. [Logical Properties](#logical-properties)
10. [Aspect Ratio](#aspect-ratio)
11. [Colors and Custom Properties](#colors-and-custom-properties)
12. [Transitions and Animations](#transitions-and-animations)
13. [Pseudo-classes and Pseudo-elements](#pseudo-classes-and-pseudo-elements)
14. [Typography](#typography)
15. [Overflow and Scrolling](#overflow-and-scrolling)
16. [CSS Functions](#css-functions)
17. [Modern CSS Features](#modern-css-features)
18. [Performance](#performance)
19. [BEM and Architecture](#bem-and-architecture)

---

# Box Model

Every element in CSS is a rectangular box. The box model defines how the size of that box is calculated.

```
┌──────────────────────────────────────────┐
│                 MARGIN                   │
│  ┌────────────────────────────────────┐  │
│  │              BORDER                │  │
│  │  ┌──────────────────────────────┐  │  │
│  │  │           PADDING            │  │  │
│  │  │  ┌────────────────────────┐  │  │  │
│  │  │  │        CONTENT         │  │  │  │
│  │  │  │   (width × height)     │  │  │  │
│  │  │  └────────────────────────┘  │  │  │
│  │  └──────────────────────────────┘  │  │
│  └────────────────────────────────────┘  │
└──────────────────────────────────────────┘
```

### box-sizing

This is the single most important box model property. It controls what `width` and `height` actually mean.

```css
/* content-box (default) — width/height applies to content only */
.box {
  box-sizing: content-box;
  width: 200px;
  padding: 20px;
  border: 5px solid black;
  /* Total width: 200 + 20 + 20 + 5 + 5 = 250px */
}

/* border-box — width/height includes padding and border */
.box {
  box-sizing: border-box;
  width: 200px;
  padding: 20px;
  border: 5px solid black;
  /* Total width: 200px (content shrinks to 150px to fit) */
}
```

Every modern project uses this reset:

```css
*, *::before, *::after {
  box-sizing: border-box;
}
```

Without this, every element's total size is unpredictable — you set `width: 200px` but the element takes up 250px because of padding and border. `border-box` makes width mean what you expect it to mean.

### Margin

```css
.box {
  margin: 20px;                    /* all sides */
  margin: 10px 20px;              /* vertical | horizontal */
  margin: 10px 20px 30px;         /* top | horizontal | bottom */
  margin: 10px 20px 30px 40px;    /* top | right | bottom | left (clockwise) */

  margin: auto;                    /* center horizontally (block elements with a width) */
  margin-inline: auto;            /* center using logical properties */
}
```

### Margin collapsing

When two vertical margins touch, they don't add up — the larger one wins. This only happens vertically, never horizontally.

```css
.box1 { margin-bottom: 30px; }
.box2 { margin-top: 20px; }
/* Gap between them: 30px (NOT 50px) — the larger margin wins */
```

**When margins collapse:**
- Adjacent siblings — top margin of one meets bottom margin of the other
- Parent and first/last child — if nothing separates them (no padding, border, or content)
- Empty blocks — top and bottom margin of the same element

```html
<!-- Parent-child collapsing -->
<div class="parent" style="margin-top: 0;">
  <div class="child" style="margin-top: 40px;">
    <!-- The child's 40px margin "leaks" out of the parent -->
  </div>
</div>

<!-- Fix: add padding, border, or overflow to the parent -->
<div class="parent" style="padding-top: 1px;">
  <div class="child" style="margin-top: 40px;">
    <!-- Now margins don't collapse — child stays inside -->
  </div>
</div>
```

**What prevents margin collapsing:**
- `overflow: hidden` (or auto/scroll) on the parent
- Padding or border on the parent (even 1px)
- Flexbox or grid containers — margins never collapse in flex/grid
- `display: flow-root` on the parent (modern, cleanest fix)

### Negative margins

```css
.box {
  margin-top: -20px;    /* pulls element UP */
  margin-left: -20px;   /* pulls element LEFT */
  margin-bottom: -20px; /* pulls NEXT element UP toward this one */
  margin-right: -20px;  /* pulls NEXT element LEFT toward this one */
}
```

### Outline vs Border

```css
/* Border — part of the box model, affects layout */
.box {
  border: 2px solid red;
  /* Element's total size increases by 4px (unless border-box) */
}

/* Outline — NOT part of the box model, doesn't affect layout */
.box {
  outline: 2px solid blue;
  outline-offset: 4px;
  /* Element size doesn't change */
}
```

Never remove outline without providing an alternative:

```css
/* BAD — removes focus indicator, breaks keyboard accessibility */
*:focus { outline: none; }

/* GOOD — custom focus indicator */
*:focus-visible {
  outline: 2px solid #4A90D9;
  outline-offset: 2px;
}
```

---

# Selectors and Specificity

### Selector types

```css
/* Universal */
* { margin: 0; }

/* Type (element) */
p { color: black; }

/* Class */
.card { padding: 16px; }
.card.featured { border: 2px solid gold; }  /* both classes on same element */

/* ID */
#header { height: 60px; }

/* Attribute */
[type="email"] { border-color: blue; }
[href^="https"] { color: green; }     /* starts with */
[href$=".pdf"] { color: red; }        /* ends with */
[href*="example"] { color: orange; }  /* contains */
[data-active] { opacity: 1; }         /* has attribute (any value) */

/* Descendant (space) — any nested level */
.card p { color: gray; }

/* Child (>) — direct children only */
.card > p { color: gray; }

/* Adjacent sibling (+) — immediately next sibling */
h2 + p { margin-top: 0; }

/* General sibling (~) — any following sibling */
h2 ~ p { color: gray; }
```

### Specificity calculation

Specificity determines which rule wins when multiple rules target the same element. Calculated as a three-part score:

```
(ID) - (Class/Attribute/Pseudo-class) - (Element/Pseudo-element)

p                    → 0-0-1
.card                → 0-1-0
#header              → 1-0-0
p.card               → 0-1-1
#header .nav a       → 1-1-1
#header .nav a:hover → 1-2-1

#header (1-0-0) beats .card.featured (0-2-0)
/* 1 ID > any number of classes */
```

**The specificity hierarchy:**

```
!important           → overrides everything (avoid)
Inline styles        → style="..." (1-0-0-0)
ID selectors         → #id
Class selectors      → .class, [attr], :pseudo-class
Element selectors    → div, p, ::pseudo-element
Universal            → * (no specificity)
```

### The cascade

When multiple rules target the same element, CSS resolves conflicts in this order:

```
1. Origin and importance
   - User agent (browser defaults)
   - User styles
   - Author styles
   - Author !important
   - User !important
   - User agent !important

2. Specificity (if same origin)

3. Source order (if same specificity — last rule wins)
```

### !important — and why to avoid it

```css
.text { color: red !important; }

#header .text { color: blue; }  /* loses to !important above */

#header .text { color: green !important; }  /* wins — same importance + higher specificity */
```

### Alternatives to !important

```css
/* 1. Double the class selector */
.button.button { color: red; }  /* specificity: 0-2-0, beats single .button */

/* 2. Use :where() for base styles (zero specificity) */
:where(.button) { color: blue; }  /* 0-0-0 */
.button { color: red; }           /* 0-1-0, easily wins */

/* 3. Use @layer for third-party CSS */
@layer vendor { .button { color: blue; } }  /* layered — low priority */
.button { color: red; }                      /* unlayered — wins */
```

### Modern pseudo-class selectors

**`:is()` — matches any, takes highest specificity of its arguments:**

```css
.card :is(h1, h2, h3) { color: navy; }

:is(#header, .nav) a { }  /* Specificity: 1-0-1 (takes #header's specificity) */
```

**`:where()` — matches any, with ZERO specificity:**

```css
:where(.card, .panel) h2 { color: navy; }  /* 0-0-1 specificity */
h2 { color: red; }  /* easily overrides */
```

**`:has()` — parent selector:**

```css
.card:has(img) { padding: 0; }
label:has(+ input:focus) { color: blue; }
body:has(.modal.open) { overflow: hidden; }
form:has(:invalid) { border-color: red; }
```

---

# Positioning

### static (default)

```css
.box {
  position: static;
  /* top, right, bottom, left, z-index have NO effect */
}
```

### relative

```css
.box {
  position: relative;
  top: 20px;
  left: 10px;
  /* Offset from NORMAL position. Original space preserved. */
  /* Creates containing block for absolute children. */
}
```

### absolute

```css
.box {
  position: absolute;
  top: 0;
  right: 0;
  /* Removed from flow. Positioned relative to nearest positioned ancestor. */
}

/* Classic pattern: absolute inside relative */
.parent { position: relative; }
.badge { position: absolute; top: -8px; right: -8px; }
```

### fixed

```css
.navbar {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  /* Positioned relative to VIEWPORT. Stays during scrolling. */
  /* ⚠️ Breaks if an ancestor has transform, filter, or perspective */
}
```

### sticky

```css
.section-header {
  position: sticky;
  top: 0;
  /* Behaves like relative until scroll threshold, then like fixed within parent */
}
```

**Sticky requires:** a threshold value (`top`, `bottom`, etc.), scrollable parent, and no `overflow: hidden/auto/scroll` on ancestors.

### Stacking context and z-index

`z-index` only works on positioned elements and flex/grid children.

**What creates a new stacking context:** `position` with `z-index`, `opacity < 1`, `transform`, `filter`, `perspective`, `clip-path`, `isolation: isolate`, `will-change`.

**The trap:** z-index only competes within the same stacking context.

```css
.parent-a { position: relative; z-index: 1; }
.parent-b { position: relative; z-index: 2; }
.child-of-a { position: relative; z-index: 9999; }
/* child-of-a still behind parent-b — parent's context is lower */
```

**Fix with `isolation`:**

```css
.component { isolation: isolate; }  /* new stacking context, no side effects */
```

### Centering techniques

```css
/* Flexbox */
.parent { display: flex; justify-content: center; align-items: center; }

/* Grid — shortest */
.parent { display: grid; place-items: center; }

/* Absolute + transform */
.child { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); }

/* Margin auto */
.child { width: 500px; margin-inline: auto; }
```

---

# Display and Layout

### Block elements

```css
/* Take full width, start on new line, respect all box model properties */
/* Examples: div, p, h1-h6, section, article, form, ul, li */
div { display: block; width: 50%; height: 100px; margin: 20px; }
```

### Inline elements

```css
/* Take only content width, don't start new line, ignore width/height */
/* Vertical margin ignored, vertical padding doesn't push elements */
/* Examples: span, a, strong, em, code, label */
span { display: inline; width: 200px; /* IGNORED */ }
```

### Inline-block

```css
/* Sits inline but respects width, height, margin, padding */
.badge { display: inline-block; width: 100px; height: 30px; margin: 10px; }
```

### display: none vs visibility: hidden

```css
/* display: none — completely removed from layout */
.hidden {
  display: none;
  /* No space taken, not accessible, not interactive */
  /* Children cannot override */
}

/* visibility: hidden — invisible but space preserved */
.invisible {
  visibility: hidden;
  /* Space still taken, not accessible, not interactive */
  /* Children CAN override with visibility: visible */
}
```

```html
<div style="visibility: hidden;">
  I'm invisible
  <span style="visibility: visible;">But I'm visible!</span>
</div>

<div style="display: none;">
  I'm gone
  <span style="display: block;">Still gone — parent removes everything</span>
</div>
```

Also consider `opacity: 0` — invisible but still takes space, still accessible, still interactive (clickable). Use for fade animations.

### display: contents

```css
.wrapper {
  display: contents;
  /* Wrapper disappears, children participate in parent's layout directly */
}
```

### Flow root

```css
.container {
  display: flow-root;
  /* New block formatting context. Contains floats. Prevents margin collapse. */
}
```

---

# Flexbox

One-dimensional layout — row OR column.

### Container properties

```css
.container {
  display: flex;

  flex-direction: row;            /* default */
  flex-direction: column;
  flex-wrap: wrap;
  flex-flow: row wrap;            /* shorthand */

  justify-content: flex-start;    /* main axis */
  justify-content: center;
  justify-content: space-between;
  justify-content: space-evenly;

  align-items: stretch;           /* cross axis (default) */
  align-items: center;
  align-items: baseline;

  align-content: center;          /* multi-line cross axis (only with wrap) */

  gap: 16px;
  row-gap: 16px;
  column-gap: 8px;
}
```

### Item properties

```css
.item {
  flex-grow: 0;      /* default: don't grow */
  flex-shrink: 1;    /* default: shrink equally */
  flex-basis: auto;  /* default: use width/height */

  flex: 0 1 auto;   /* default (grow shrink basis) */
  flex: 1;           /* = 1 1 0: grow equally, ignore content size */
  flex: auto;        /* = 1 1 auto: grow equally, respect content size */
  flex: none;        /* = 0 0 auto: rigid */

  align-self: center;
  order: -1;         /* move to start */
}
```

### Common patterns

```css
/* Navbar: logo left, links right */
.navbar { display: flex; justify-content: space-between; align-items: center; }

/* Footer at bottom */
body { display: flex; flex-direction: column; min-height: 100vh; }
main { flex: 1; }

/* Equal height cards */
.card-grid { display: flex; gap: 16px; }
.card { flex: 1; display: flex; flex-direction: column; }
.card-body { flex: 1; }
.card-footer { margin-top: auto; }

/* Push last item right */
.item:last-child { margin-left: auto; }
```

### Gotchas

```css
/* Text truncation needs min-width: 0 in flex items */
.item {
  flex: 1;
  min-width: 0;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
```

---

# CSS Grid

Two-dimensional layout — rows AND columns.

### Container

```css
.grid {
  display: grid;

  grid-template-columns: 200px 1fr 200px;
  grid-template-columns: repeat(3, 1fr);
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));  /* responsive */

  grid-template-rows: 60px 1fr 40px;
  grid-auto-rows: 150px;        /* height of auto-created rows */
  grid-auto-flow: dense;        /* fill gaps by reordering */

  gap: 16px;

  justify-items: center;        /* horizontal alignment of items in cells */
  align-items: center;          /* vertical alignment of items in cells */
  place-items: center;          /* both */

  justify-content: center;      /* horizontal alignment of the grid itself */
  align-content: center;        /* vertical alignment of the grid itself */
}
```

### Template areas

```css
.layout {
  display: grid;
  grid-template-columns: 200px 1fr 200px;
  grid-template-rows: 60px 1fr 40px;
  grid-template-areas:
    "header  header  header"
    "sidebar content aside"
    "footer  footer  footer";
}

.header  { grid-area: header; }
.sidebar { grid-area: sidebar; }
.content { grid-area: content; }
.footer  { grid-area: footer; }

@media (max-width: 768px) {
  .layout {
    grid-template-columns: 1fr;
    grid-template-areas: "header" "content" "sidebar" "aside" "footer";
  }
}
```

### Item placement

```css
.item {
  grid-column: 1 / 3;       /* line 1 to line 3 (spans 2 columns) */
  grid-column: span 2;      /* spans 2 from wherever placed */
  grid-row: 1 / 2;
  grid-area: 1 / 1 / 3 / 3; /* row-start / col-start / row-end / col-end */
  justify-self: center;
  align-self: end;
}
```

### auto-fill vs auto-fit

```css
/* auto-fill: creates empty tracks if space allows */
grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));

/* auto-fit: collapses empty tracks, items stretch to fill */
grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
```

Use `auto-fit` for most responsive grids.

### Subgrid

```css
.child {
  grid-column: span 2;
  display: grid;
  grid-template-columns: subgrid;  /* inherits parent's column tracks */
}
```

### Common patterns

```css
/* Responsive card grid — no media queries */
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 24px;
}

/* Full bleed within constrained layout */
.content {
  display: grid;
  grid-template-columns: 1fr min(65ch, 100%) 1fr;
}
.content > * { grid-column: 2; }
.full-bleed { grid-column: 1 / -1; }
```

### Flexbox vs Grid

```
Flexbox: one dimension, content-driven, navbars, centering, input groups
Grid: two dimensions, layout-driven, page layouts, card grids, dashboards
```

Use both together — grid for page layout, flex for component internals.

---

# Responsive Design

### Media queries

```css
@media (max-width: 768px) { }
@media (min-width: 769px) { }
@media (width <= 768px) { }                    /* range syntax (modern) */
@media (768px <= width <= 1024px) { }

@media (hover: hover) { }                      /* has hover */
@media (hover: none) { }                       /* touch device */
@media (prefers-color-scheme: dark) { }
@media (prefers-reduced-motion: reduce) { }
@media (pointer: coarse) { }                   /* imprecise (touch) */
@media (pointer: fine) { }                     /* precise (mouse) */
```

### Mobile-first (recommended)

```css
.card { padding: 16px; }                       /* base: mobile */
@media (min-width: 768px) { .card { padding: 24px; display: flex; } }
@media (min-width: 1200px) { .card { padding: 32px; max-width: 1200px; } }
```

### Container queries

```css
.card-container {
  container-type: inline-size;
  container-name: card;
}

@container card (min-width: 400px) {
  .card { display: flex; gap: 16px; }
}
```

### Fluid typography

```css
h1 { font-size: clamp(1.5rem, 4vw, 3rem); }
.section { padding: clamp(1rem, 5vw, 4rem); }
.container { width: clamp(300px, 90vw, 1200px); margin-inline: auto; }
```

---

# Units

### Absolute

```css
width: 200px;  /* most common — avoid cm, mm, in, pt for screens */
```

### Relative

```css
font-size: 1.5em;    /* relative to element's (or parent's for font-size) font-size */
font-size: 1.5rem;   /* relative to root font-size (default 16px) → 24px */
width: 50%;           /* relative to parent */
max-width: 65ch;      /* ~65 characters per line — ideal for readability */
```

### Viewport units

```css
height: 100vh;        /* 100% of viewport height */
width: 100vw;         /* 100% of viewport width */
font-size: 5vmin;     /* 5% of smaller dimension */
font-size: 5vmax;     /* 5% of larger dimension */

/* New units — solve mobile address bar problem */
height: 100dvh;       /* dynamic: updates when address bar shows/hides */
height: 100svh;       /* small: viewport WITH address bar (smallest) */
height: 100lvh;       /* large: viewport WITHOUT address bar (largest) */
```

**The mobile viewport problem:** `100vh` equals the large viewport (bar hidden). When bar is visible, content overflows. Fix:

```css
.hero {
  height: 100vh;    /* fallback */
  height: 100dvh;   /* override for supporting browsers */
}
```

### When to use which

```
px    — borders, shadows, small fixed values, media queries
rem   — font sizes, spacing, widths — scales with root
em    — component-internal spacing that scales with component's font
%     — fluid widths relative to parent
vw/vh — full-page sections, viewport-relative sizing
dvh   — full-height sections on mobile
ch    — max-width for readable text
fr    — grid track sizing
```

---

# Logical Properties

Traditional CSS uses physical directions (top, right, bottom, left). Logical properties use flow-relative directions (block = vertical, inline = horizontal in English). This matters for RTL languages.

```css
/* Physical → Logical */
margin-top         → margin-block-start
margin-bottom      → margin-block-end
margin-left        → margin-inline-start
margin-right       → margin-inline-end
/* Shorthand: */
margin: 10px 20px  → margin-block: 10px; margin-inline: 20px;

width              → inline-size
height             → block-size
min-width          → min-inline-size

top                → inset-block-start
left               → inset-inline-start
/* Shorthand: */
top: 0; right: 0; bottom: 0; left: 0;  →  inset: 0;

text-align: left   → text-align: start
text-align: right  → text-align: end
```

### Practical examples

```css
.container {
  max-inline-size: 1200px;
  margin-inline: auto;
}

.card {
  padding-block: 16px;
  padding-inline: 24px;
  border-inline-start: 4px solid blue;
}

.overlay {
  position: absolute;
  inset: 0;  /* replaces top/right/bottom/left: 0 */
}
```

Even without RTL, logical properties are worth using: `margin-inline: auto` replaces two properties, `inset: 0` replaces four.

---

# Aspect Ratio

```css
.video { aspect-ratio: 16 / 9; width: 100%; }
.square { aspect-ratio: 1; width: 200px; }
.portrait { aspect-ratio: 3 / 4; }
```

```
Ratio   | Use case
--------|---------------------------
1 / 1   | Profile pictures, icons
16 / 9  | Video embeds, hero images
4 / 3   | Classic photo format
3 / 2   | DSLR photo format
21 / 9  | Ultra-wide banners
```

If content exceeds the ratio, the box grows. Add `overflow: hidden` to clip.

---

# Colors and Custom Properties

### Color formats

```css
color: #ff0000;                        /* hex */
color: #f00;                           /* shorthand hex */
color: #ff000080;                      /* hex with alpha */
color: rgb(255 0 0);                   /* modern RGB */
color: rgb(255 0 0 / 50%);            /* with alpha */
color: hsl(0 100% 50%);               /* HSL */
color: hsl(0 100% 50% / 50%);         /* with alpha */
color: oklch(70% 0.15 30);            /* perceptually uniform */
```

**HSL is intuitive:** Hue (0°=red, 120°=green, 240°=blue), Saturation (0%=gray, 100%=vivid), Lightness (0%=black, 50%=pure, 100%=white).

```css
:root {
  --primary-h: 220;
  --primary-s: 90%;
  --primary: hsl(var(--primary-h) var(--primary-s) 50%);
  --primary-light: hsl(var(--primary-h) var(--primary-s) 70%);
  --primary-dark: hsl(var(--primary-h) var(--primary-s) 30%);
}
```

### CSS Variables

```css
:root {
  --color-primary: #4A90D9;
  --spacing-md: 16px;
  --radius: 8px;
}

.card {
  color: var(--color-primary);
  padding: var(--spacing-md);
  border-radius: var(--radius);
  margin: var(--margin, 20px);  /* fallback if undefined */
}
```

### Scoping and dark mode

```css
:root { --bg: white; --text: black; }

@media (prefers-color-scheme: dark) {
  :root { --bg: #1a1a2e; --text: #e0e0e0; }
}

.card { background: var(--bg); color: var(--text); }
```

### accent-color — one-line form control styling

```css
:root { accent-color: #4A90D9; }
/* Applies to checkboxes, radio buttons, range sliders, progress bars */
/* Browser auto-handles foreground contrast */
```

### color-scheme — native dark mode

```css
:root { color-scheme: light dark; }
/* Browser chrome, scrollbars, form controls adapt automatically */
/* Combine with prefers-color-scheme for custom styles */
```

```html
<meta name="color-scheme" content="light dark" />
```

### CSS vs Sass variables

```
Feature              | CSS Variables     | Sass Variables
---------------------|-------------------|------------------
Evaluated at         | Runtime (browser) | Compile time
Changed by JS        | ✅                | ❌
Scoped/inherited     | ✅                | ❌
Need build step      | ❌                | ✅
```

```javascript
document.documentElement.style.setProperty('--color-primary', '#ff0000');
```

---

# Transitions and Animations

### Transitions

```css
.button {
  background: #4A90D9;
  transition: background 0.3s ease, transform 0.2s ease;
  /* Prefer specific properties over 'all' for performance */
}
.button:hover {
  background: #357ABD;
  transform: scale(1.05);
}
```

### Timing functions

```css
transition-timing-function: linear;
transition-timing-function: ease;          /* default */
transition-timing-function: ease-in;
transition-timing-function: ease-out;
transition-timing-function: ease-in-out;
transition-timing-function: cubic-bezier(0.68, -0.55, 0.265, 1.55);  /* bounce */
```

### What CAN'T be transitioned

`display`, `font-family`, `position`, `float`, `content`. Common fix for display:

```css
.modal {
  visibility: hidden; opacity: 0;
  transition: opacity 0.3s, visibility 0.3s;
}
.modal.active {
  visibility: visible; opacity: 1;
}
```

### Keyframe animations

```css
@keyframes slideIn {
  from { transform: translateX(-100%); opacity: 0; }
  to { transform: translateX(0); opacity: 1; }
}

.element {
  animation: slideIn 0.5s ease-out;
  animation-fill-mode: forwards;    /* keeps end state */
  animation-iteration-count: infinite;
  animation-direction: alternate;
}
```

### Transform

```css
transform: translateX(50px);
transform: scale(1.5);
transform: rotate(45deg);
transform: skewX(10deg);
transform: translateX(50px) rotate(45deg) scale(1.2);  /* order matters */
transform-origin: top left;
```

### Performance

```css
/* CHEAP: transform, opacity (compositor only) */
/* EXPENSIVE: width, height, margin, padding, top, left (trigger layout) */

/* BAD */
.box:hover { width: 200px; left: 20px; }
/* GOOD */
.box:hover { transform: translateX(20px) scale(1.1); }
```

### will-change

```css
.card:hover { will-change: transform; }  /* apply temporarily before animation */
/* DON'T: * { will-change: transform; } — wastes GPU memory */
```

### Reduced motion

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

---

# Pseudo-classes and Pseudo-elements

### Pseudo-classes (state)

```css
/* Interaction */
a:hover { }  a:active { }  a:visited { }
button:focus { }  button:focus-visible { }  button:focus-within { }

/* Form */
input:required { }  input:valid { }  input:invalid { }
input:placeholder-shown { }  input:checked { }  input:disabled { }
input:read-only { }  input:autofill { }

/* Structural */
li:first-child { }  li:last-child { }
li:nth-child(3) { }  li:nth-child(odd) { }  li:nth-child(3n+1) { }
li:nth-last-child(2) { }
p:first-of-type { }  div:only-child { }
:root { }  :empty { }

/* Matching */
:not(.active) { }  :is(h1, h2, h3) { }  :where(h1, h2) { }  :has(img) { }
:target { }  :lang(en) { }
```

### :focus vs :focus-visible

```css
button:focus { outline: none; }           /* remove for mouse clicks */
button:focus-visible { outline: 2px solid #4A90D9; outline-offset: 2px; }  /* keyboard only */
```

### :nth-child advanced

```css
li:nth-child(-n+5) { }     /* first 5 */
li:nth-child(n+4) { }      /* after 3rd */
li:nth-child(n+3):nth-child(-n+7) { }  /* 3 through 7 */
```

### Pseudo-elements (parts)

```css
.quote::before { content: "\201C"; font-size: 2rem; }
.divider::after { content: ""; display: block; height: 1px; background: #ddd; }

input::placeholder { color: #999; font-style: italic; }
::selection { background: #4A90D9; color: white; }
p::first-letter { font-size: 3rem; float: left; }
li::marker { color: #4A90D9; }
```

### Practical patterns

```css
label.required::after { content: " *"; color: red; }
a[href^="http"]::after { content: " ↗"; font-size: 0.8em; }

/* Custom checkbox */
input[type="checkbox"] { appearance: none; width: 20px; height: 20px; border: 2px solid #ccc; }
input[type="checkbox"]:checked::after { content: "✓"; text-align: center; color: #4A90D9; }
```

---

# Typography

### Font loading

```css
body {
  font-family: system-ui, -apple-system, BlinkMacSystemFont,
    'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
}

@font-face {
  font-family: 'MyFont';
  src: url('/fonts/myfont.woff2') format('woff2');
  font-weight: 400;
  font-display: swap;  /* show fallback immediately, swap when loaded */
}
```

### font-display

```
auto     — browser decides
block    — invisible text up to 3s, then swap (FOIT)
swap     — show fallback immediately, swap when ready (FOUT) ← recommended for body
fallback — short invisible period, then fallback
optional — show fallback, only use font if cached ← best for avoiding layout shift
```

### Line height

```css
body { line-height: 1.5; }  /* unitless preferred — inherits multiplier, not computed value */
h1 { line-height: 1.2; }    /* tighter for headings */
```

### Text overflow

```css
/* Single line */
.truncate { white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }

/* Multi-line */
.truncate-multi {
  display: -webkit-box; -webkit-box-orient: vertical;
  -webkit-line-clamp: 3; overflow: hidden;
}

/* Word breaking */
.long-text { overflow-wrap: break-word; hyphens: auto; }
```

### Other properties

```css
letter-spacing: 0.05em;
text-transform: uppercase;
text-decoration: underline wavy red;
text-underline-offset: 4px;
white-space: pre-wrap;
font-variant-numeric: tabular-nums;  /* equal-width numbers */
```

---

# Overflow and Scrolling

```css
overflow: visible;  /* default */
overflow: hidden;   /* clips, creates scroll container (scrollable via JS) */
overflow: clip;     /* clips, NO scroll container (truly clips) */
overflow: auto;     /* scrollbar only when needed */
overflow: scroll;   /* always shows scrollbar */
```

### Scroll behavior

```css
html { scroll-behavior: smooth; }

@media (prefers-reduced-motion: reduce) {
  html { scroll-behavior: auto; }
}
```

### Scroll snap

```css
.carousel {
  display: flex; overflow-x: auto;
  scroll-snap-type: x mandatory;
  scroll-padding: 0 20px;
}
.carousel-item {
  scroll-snap-align: start;
  flex: 0 0 100%;
}
```

### Overscroll behavior

```css
.modal-body { overflow-y: auto; overscroll-behavior: contain; }  /* prevent scroll leak */
body { overscroll-behavior-y: none; }  /* prevent pull-to-refresh */
```

### Scrollbar styling

```css
/* Webkit */
.container::-webkit-scrollbar { width: 8px; }
.container::-webkit-scrollbar-track { background: #f0f0f0; }
.container::-webkit-scrollbar-thumb { background: #888; border-radius: 4px; }

/* Standard */
.container { scrollbar-width: thin; scrollbar-color: #888 #f0f0f0; }

/* Hide scrollbar */
.container { scrollbar-width: none; }
.container::-webkit-scrollbar { display: none; }
```

---

# CSS Functions

### calc()

```css
width: calc(100% - 250px);
height: calc(100vh - 60px);
/* ⚠️ Spaces around operators REQUIRED */
```

### min(), max(), clamp()

```css
width: min(90vw, 1200px);          /* smaller value */
width: max(300px, 50vw);           /* larger value */
width: clamp(300px, 90vw, 1200px); /* min, preferred, max */
font-size: clamp(1rem, 2.5vw, 2rem);
```

### var()

```css
padding: var(--spacing, 16px);     /* with fallback */
color: var(--theme-color, var(--color, black));  /* nested fallback */
```

### env()

```css
padding: env(safe-area-inset-top, 20px);  /* iPhone notch safe area */
```

### Gradients

```css
background: linear-gradient(135deg, #ff0000, #0000ff);
background: radial-gradient(circle at center, #ff0000, transparent);
background: conic-gradient(red 0deg 90deg, blue 90deg 180deg, green 180deg 360deg);
background: repeating-linear-gradient(45deg, #f0f0f0, #f0f0f0 10px, #fff 10px, #fff 20px);

/* Gradient text */
.gradient-text {
  background: linear-gradient(135deg, #667eea, #764ba2);
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
}

/* Image overlay */
.hero {
  background: linear-gradient(rgba(0,0,0,0.5), rgba(0,0,0,0.7)), url('hero.jpg') center/cover;
}
```

### Other functions

```css
clip-path: circle(50%);
clip-path: polygon(0 0, 100% 0, 100% 100%);
filter: blur(4px) brightness(1.2) grayscale(100%);
backdrop-filter: blur(10px);
background: color-mix(in srgb, #ff0000, #0000ff 50%);
```

---

# Modern CSS Features

### @layer — cascade layers

```css
@layer reset, base, components, utilities;

@layer reset { * { margin: 0; } }
@layer base { a { color: blue; } }
@layer components { .nav a { color: white; } }
/* Layer order determines winner, not specificity */
/* Unlayered styles beat ALL layers */
```

### CSS Nesting

```css
.card {
  padding: 16px;
  .title { font-size: 1.5rem; }
  &:hover { box-shadow: 0 2px 8px rgba(0,0,0,0.1); }
  &.featured { border-color: gold; }
  @media (max-width: 768px) { padding: 8px; }
}
```

### @scope

```css
@scope (.card) {
  h2 { font-size: 1.5rem; }
  p { color: gray; }
}

@scope (.card) to (.card-footer) {
  /* Styles inside .card but NOT inside .card-footer */
}
```

### color-mix()

```css
.button:hover { background: color-mix(in srgb, var(--base-color), black 20%); }
.button.light { background: color-mix(in srgb, var(--base-color), white 30%); }
```

### @starting-style — transition from display: none

```css
.modal {
  display: none; opacity: 0;
  transition: opacity 0.3s, display 0.3s allow-discrete;
  @starting-style { opacity: 0; }
}
.modal.open { display: block; opacity: 1; }
```

### @supports — feature detection

```css
@supports (aspect-ratio: 1) {
  .card-image { aspect-ratio: 16 / 9; }
}
@supports not (aspect-ratio: 1) {
  .card-image { padding-top: 56.25%; }
}
@supports selector(:has(*)) {
  .card:has(img) { padding: 0; }
}
```

### View transitions

```css
::view-transition-old(root) { animation: fade-out 0.3s; }
::view-transition-new(root) { animation: fade-in 0.3s; }
```

---

# Performance

### Repaint vs Reflow vs Composite

```
Most expensive ←————→ Least expensive
Reflow (Layout) → Repaint → Composite

Reflow:    width, height, margin, padding, font-size, reading offsetHeight
Repaint:   color, background, visibility, box-shadow
Composite: transform, opacity (cheapest — GPU only)
```

### Layout thrashing

```javascript
// BAD — forces reflow each iteration
for (const el of elements) {
  el.style.width = container.offsetWidth + 'px';
}

// GOOD — batch reads, then writes
const width = container.offsetWidth;
for (const el of elements) {
  el.style.width = width + 'px';
}
```

### contain

```css
.card { contain: content; }  /* layout + paint isolation */
```

### content-visibility

```css
.section {
  content-visibility: auto;
  contain-intrinsic-size: 0 500px;
  /* Skips rendering off-screen content — massive perf win for long pages */
}
```

### will-change

```css
/* DO: apply temporarily before animation */
.card:hover { will-change: transform; }
/* DON'T: apply globally or permanently */
```

### Critical CSS

```html
<head>
  <style>/* inline above-the-fold CSS */</style>
  <link rel="preload" href="styles.css" as="style"
    onload="this.onload=null;this.rel='stylesheet'" />
</head>
```

### Checklist

```
Animate only transform/opacity     — avoids reflow/repaint
Use contain: content               — isolates reflow
content-visibility: auto           — skips off-screen rendering
will-change (sparingly)            — pre-promotes to GPU
Reduce DOM depth                   — faster style calculation
Batch reads and writes             — avoids layout thrashing
Inline critical CSS                — faster first paint
```

---

# BEM and Architecture

### BEM (Block Element Modifier)

```css
.card { }                 /* Block */
.card__title { }          /* Element (double underscore) */
.card__body { }
.card--featured { }       /* Modifier (double hyphen) */
.card__title--large { }
```

```html
<article class="card card--featured">
  <h2 class="card__title card__title--large">Featured</h2>
  <p class="card__body">Content</p>
</article>
```

Every selector is one class — specificity is always 0-1-0. No nesting conflicts.

### Architecture approaches

**Tailwind (utility-first):**
```html
<article class="p-4 border rounded-lg bg-yellow-50">
  <h2 class="text-xl font-bold">Post</h2>
</article>
```

**CSS-in-JS (styled-components):**
```javascript
const Card = styled.article`padding: 16px; border-radius: 8px;`;
```

**CSS Modules:**
```css
/* Card.module.css */
.card { padding: 16px; }
```
```javascript
import styles from './Card.module.css';
<article className={styles.card}>
```

### Choosing

```
BEM           — large teams, long-lived projects
Tailwind      — rapid development, small teams
CSS-in-JS     — React apps, dynamic theming
CSS Modules   — component-based apps, no runtime cost
@layer        — mixing third-party + custom CSS
```
