## Table of Contents

### Foundations (study first)

1. [Box Model](#box-model)
2. [Cascade, Specificity & Keywords](#cascade-specificity--keywords)
3. [Formatting Contexts](#formatting-contexts)
4. [Containing Block, Positioning & Stacking](#containing-block-positioning--stacking)
5. [Intrinsic Sizing](#intrinsic-sizing)

### Layout

6. [Display and Layout](#display-and-layout)
7. [Flexbox](#flexbox)
8. [CSS Grid](#css-grid)

### Responsive system

9. [Responsive Design](#responsive-design)
10. [Units](#units)
11. [Logical Properties](#logical-properties)
12. [Aspect Ratio](#aspect-ratio)

### Visual & interaction

13. [Colors and Custom Properties](#colors-and-custom-properties)
14. [Transitions and Animations](#transitions-and-animations)
15. [Pseudo-classes and Pseudo-elements](#pseudo-classes-and-pseudo-elements)
16. [Typography](#typography)
17. [Overflow and Scrolling](#overflow-and-scrolling)
18. [CSS Functions](#css-functions)

### Senior layer

19. [Modern CSS Features](#modern-css-features)
20. [Performance](#performance)
21. [Architecture](#architecture)

---

# Box Model

**Confusion:** You set `width: 200px` but the element is wider. Or two stacked blocks have less gap than `margin-bottom + margin-top`.

**Model:** Every element is a rectangle made of content → padding → border → margin. `width`/`height` mean different things depending on `box-sizing`. Vertical margins of adjoining blocks **collapse** into one; horizontal margins never do.

**Rule:** Use `border-box` globally. Prefer padding/gap/BFC for spacing control when collapse surprises you. Never remove focus outlines without `:focus-visible` replacement.

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

```css
/* content-box (default) — width applies to content only */
.box {
  box-sizing: content-box;
  width: 200px;
  padding: 20px;
  border: 5px solid black;
  /* Total: 200 + 40 + 10 = 250px */
}

/* border-box — width includes padding + border */
.box {
  box-sizing: border-box;
  width: 200px;
  padding: 20px;
  border: 5px solid black;
  /* Total: 200px (content shrinks) */
}

*, *::before, *::after {
  box-sizing: border-box;
}
```

### Margin collapsing

When two vertical margins touch, the **larger** wins — they do not add.

```css
.box1 { margin-bottom: 30px; }
.box2 { margin-top: 20px; }
/* Gap: 30px, not 50px */
```

**Collapses when:**
- Adjacent siblings (top meets bottom)
- Parent and first/last child (nothing between them)
- Empty blocks (own top + bottom)

**Prevents collapse:**
- Padding or border on the parent (even 1px)
- `overflow` other than `visible` on the parent
- Flex or grid container (margins never collapse inside)
- `display: flow-root` (cleanest modern fix)

```css
.parent { display: flow-root; } /* new BFC — child margins stay inside */
```

### Outline vs border

```css
/* Border — part of the box, affects layout */
.box { border: 2px solid red; }

/* Outline — outside the box, no layout impact */
.box {
  outline: 2px solid blue;
  outline-offset: 4px;
}

/* BAD */
*:focus { outline: none; }

/* GOOD */
*:focus-visible {
  outline: 2px solid #4A90D9;
  outline-offset: 2px;
}
```

### Self-check

1. Why does `width: 100%` + `padding: 16px` overflow a parent under `content-box`?
2. Name three ways to stop parent/child margin collapse.
3. Why is `outline` safer than `border` for focus rings?

---

# Cascade, Specificity & Keywords

**Confusion:** A rule “should win” but doesn’t — or `!important` wars start. Or you reset a property and something unexpected comes back.

**Model:** Conflict resolution is ordered: **origin/importance → layers → specificity → source order**. Specificity is `(inline) / IDs / classes+attrs+pseudo-classes / elements+pseudo-elements`. Keywords control *which value* you pull from that cascade: `inherit`, `initial`, `unset`, `revert`, `revert-layer`.

**Rule:** Prefer lower specificity and `@layer` over `!important`. Use `:where()` for zero-specificity bases. Know the five keywords cold for interviews.

### Specificity

```
(ID) - (Class/Attribute/Pseudo-class) - (Element/Pseudo-element)

p                    → 0-0-1
.card                → 0-1-0
#header              → 1-0-0
#header .nav a:hover → 1-2-1

/* 1 ID beats any number of classes */
```

```
!important (by origin)  → strongest lever (avoid in app CSS)
Inline styles           → style="..."
ID                      → #id
Class / attr / :pseudo  → .class, [attr], :hover
Element / ::pseudo      → div, ::before
Universal / :where()    → 0 specificity
```

### Cascade order (simplified author view)

```
1. Origin + importance (!important flips user/author ranking)
2. @layer order (earlier layers lose to later; unlayered beats layered)
3. Specificity
4. Source order (last wins)
```

### Selector essentials

```css
.card > p { }           /* direct child */
h2 + p { }              /* adjacent sibling */
h2 ~ p { }              /* general sibling */
[href^="https"] { }     /* starts with */
[href$=".pdf"] { }      /* ends with */
[data-active] { }       /* attribute present */

:is(#header, .nav) a { }     /* takes highest specificity of args */
:where(.card, .panel) h2 { } /* always 0-0-0 for the :where part */
.card:has(img) { }           /* parent selector */
```

### Cascade keywords (P0)

```css
color: inherit;       /* parent’s computed value */
color: initial;       /* property’s initial value (often not what you want) */
color: unset;         /* inherit if inherited property, else initial */
display: revert;      /* roll back to user-agent (or user) style — “undo my author CSS” */
color: revert-layer;  /* roll back to previous @layer only */
```

| Keyword        | Means                                      | Typical use                          |
|----------------|--------------------------------------------|--------------------------------------|
| `inherit`      | Take parent’s value                        | Force inheritance (`display` etc.) |
| `initial`      | Spec initial for that property             | Rare; often too aggressive           |
| `unset`        | Inherit *or* initial intelligently         | Soft reset of one property           |
| `revert`       | Undo author styles toward browser default  | Unstyle a component island           |
| `revert-layer` | Undo within cascade layers                 | Escape a utility layer cleanly       |

```css
@layer reset, base, components, utilities;

@layer reset { * { margin: 0; } }
@layer base { a { color: blue; } }
@layer components { .nav a { color: white; } }
/* Unlayered styles beat ALL layers */

/* Escape hatch without !important */
.prose a { color: revert-layer; }
```

### Alternatives to !important

```css
.button.button { }              /* double class: 0-2-0 */
:where(.button) { color: blue; } /* 0 specificity base */
.button { color: red; }          /* easily wins */

@layer vendor { .button { color: blue; } }
.button { color: red; }          /* unlayered wins */
```

### Self-check

1. Why does `#header` beat `.a.b.c.d.e`?
2. Difference between `unset` and `revert`?
3. Does an unlayered rule beat a more specific rule inside `@layer components`?

---

# Formatting Contexts

**Confusion:** Floats poke out of parents. Margins “leak” from a child. `overflow: hidden` “fixes” things for mysterious reasons.

**Model:** A **formatting context** is a layout arena. Block layout happens in a **BFC** (Block Formatting Context). Flex/Grid create their own contexts (FFC/GFC). Inside a new BFC: floats are contained, margins don’t collapse through the boundary, and in-flow boxes lay out independently of outside floats.

**Rule:** When you need “contain this layout,” create a BFC on purpose (`flow-root`, flex, grid, or non-visible `overflow`) — don’t cargo-cult `overflow: hidden`.

### What creates a BFC

```css
.el {
  display: flow-root;     /* purpose-built — prefer this */
  display: flex;          /* also a flex formatting context */
  display: grid;
  float: left;
  position: absolute;     /* or fixed */
  overflow: auto;         /* any value except visible */
  contain: layout;        /* or content / strict */
}
```

### What a BFC buys you

```css
/* 1. Contain floats */
.clearfix {
  display: flow-root;
}

/* 2. Stop margin collapse through parent edge */
.card {
  display: flow-root;
}

/* 3. Ignore outside floats (new context beside a float) */
.beside-float {
  display: flow-root;
}
```

### Floats in one sentence

Floats are taken out of normal flow but still affect line boxes. A BFC parent expands to enclose its floats — that is float containment.

### Self-check

1. Why does `display: flow-root` fix margin leak without clipping overflow?
2. Do flex item margins collapse with each other? With the flex container’s margins?
3. Name two ways to create a BFC that don’t use `overflow`.

---

# Containing Block, Positioning & Stacking

**Confusion:** `position: absolute; top: 0` sticks to the wrong ancestor. `position: fixed` scrolls away. `z-index: 9999` still sits behind something.

**Model:**
1. **Containing block** — the rectangle offsets (`top`/`left`/…) are resolved against. It is *not* always “nearest positioned ancestor.”
2. **Position modes** change participation in flow and which containing block is used.
3. **Stacking contexts** nest; `z-index` only competes **inside** the same context.

**Rule:** Trace containing block first, then stacking context. Use `isolation: isolate` to create a clean local stacking root.

### Containing block rules (P0)

| Situation | Containing block for absolute |
|-----------|-------------------------------|
| Ancestor with `position` ≠ `static` | That ancestor’s padding edge |
| Ancestor with `transform`, `filter`, `perspective`, `will-change: transform`, `contain: paint`, `backdrop-filter` | That ancestor (even if `position: static`) |
| None of the above | Initial containing block (viewport-related) |

**Fixed:** normally the viewport — **unless** an ancestor creates a containing block via `transform` / `filter` / `perspective` / etc. Then “fixed” acts like absolute inside that ancestor.

```css
/* Classic absolute-in-relative */
.parent { position: relative; }
.badge { position: absolute; top: -8px; right: -8px; }

/* The trap: transform on ancestor breaks fixed-to-viewport */
.modal-root { transform: translateZ(0); } /* fixed children now trapped */
```

### Position modes

```css
/* static — default; top/left/z-index ignored */
/* relative — offset from normal position; space preserved; CB for abs kids */
/* absolute — out of flow; CB = positioned/transformed ancestor */
/* fixed — out of flow; CB = viewport (unless transformed ancestor) */
/* sticky — relative until threshold, then stuck within ancestor */
```

**Sticky needs:** a threshold (`top` etc.), a scroll ancestor, and no `overflow: hidden|auto|scroll` on ancestors between it and the scrollport (common trap).

### Stacking contexts

`z-index` works on positioned elements and flex/grid children.

**Creates a new stacking context (common list):** `position` + `z-index` other than `auto`, `opacity < 1`, `transform`, `filter`, `perspective`, `clip-path`, `isolation: isolate`, `will-change`, `contain: paint`.

```css
.parent-a { position: relative; z-index: 1; }
.parent-b { position: relative; z-index: 2; }
.child-of-a { position: relative; z-index: 9999; }
/* child-of-a still behind parent-b — nested context loses */

.component { isolation: isolate; } /* local stacking root, few side effects */
```

### Centering

```css
.parent { display: flex; justify-content: center; align-items: center; }
.parent { display: grid; place-items: center; }
.child { position: absolute; inset: 50% auto auto 50%; transform: translate(-50%, -50%); }
.child { width: 500px; margin-inline: auto; }
```

### Self-check

1. Why can `position: fixed` fail inside a CSS-transformed modal?
2. Why doesn’t `z-index: 9999` beat a parent with `z-index: 2` sibling?
3. What three conditions does sticky need?

---

# Intrinsic Sizing

**Confusion:** Flex/grid items refuse to shrink, text won’t truncate, or `1fr` behaves like content-sized tracks.

**Model:** Boxes have **preferred** and **minimum** content sizes (`min-content`, `max-content`, `fit-content`). In flex/grid, the **automatic minimum size** often floors at `min-content`, so items won’t shrink below their longest word/image unless you override (`min-width: 0` / `min-height: 0`).

**Rule:** When something won’t shrink or truncate inside flex/grid, set `min-width: 0` (or `overflow: hidden`) on the item. Prefer `minmax(0, 1fr)` over bare `1fr` when tracks must be allowed to shrink.

### Keywords

```css
width: min-content;   /* shrink-wrap to longest unbreakable piece */
width: max-content;   /* expand to fit content without wrapping */
width: fit-content;   /* min(max-content, available) roughly */
width: fit-content(200px);
```

### The flex/grid shrink floor

```css
/* Default-ish mental model: min-width: auto ≈ min-content */
.item {
  flex: 1;
  min-width: 0;              /* allow shrink below content size */
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.grid {
  grid-template-columns: minmax(0, 1fr) minmax(0, 1fr);
  /* bare 1fr is often minmax(auto, 1fr) — auto won’t go below min-content */
}
```

### `fr` vs `auto` vs `%`

```
auto     — size to content (intrinsic)
1fr      — share free space (flexible); min often still auto
50%      — percentage of the grid container / CB
minmax(0, 1fr) — flexible AND allowed to shrink to zero
```

### Self-check

1. Why does `flex: 1` alone often fail to ellipsize text?
2. What’s the difference between `1fr` and `minmax(0, 1fr)`?
3. When would you use `width: min-content` on purpose?

---

# Display and Layout

**Confusion:** `width` on a `span` does nothing. `display: none` vs `visibility` vs `opacity` get mixed up. A wrapper breaks flex/grid participation.

**Model:** `display` chooses the element’s **outer** role (block vs inline level) and **inner** layout mode (flow, flex, grid, …). Outer role controls how it sits among siblings; inner mode controls how children are laid out.

**Rule:** Use `inline-block` only when you need inline flow + box dimensions. Prefer flex/grid for real UI layout. Use `contents` carefully (a11y caveats in some browsers historically).

### Outer roles

```css
/* block — new line, full width by default, full box model */
div { display: block; }

/* inline — shares line, ignores width/height; vertical margin doesn’t push */
span { display: inline; width: 200px; /* ignored */ }

/* inline-block — inline placement + respects width/height/margin */
.badge { display: inline-block; width: 100px; height: 30px; }
```

### Hide semantics

```css
display: none;       /* removed from layout + a11y tree; kids can’t override */
visibility: hidden;  /* space kept; kids CAN set visibility: visible */
opacity: 0;          /* space kept; still interactive + accessible — use for fades */
```

```css
.wrapper { display: contents; } /* children join grandparent’s layout */
.container { display: flow-root; } /* see Formatting Contexts */
```

### Self-check

1. Why is `opacity: 0` wrong for “remove from tab order”?
2. What does `display: contents` do to the box tree?
3. Block vs inline: which honors `width`?

---

# Flexbox

**Confusion:** Equal columns aren’t equal. Footer won’t stick down. Truncation fails. `flex: 1` vs `flex: auto` surprise you.

**Model:** One-dimensional distribution along a **main axis** (grow/shrink/basis) and alignment on the **cross axis**. Default `flex: 0 1 auto`. Automatic minimum size (see [Intrinsic Sizing](#intrinsic-sizing)) often blocks shrinking.

**Rule:** Decide grow/shrink/basis explicitly. For equal flexible children that must shrink, use `flex: 1; min-width: 0` (or `flex: 1 1 0`).

### Container

```css
.container {
  display: flex;
  flex-direction: row;           /* main axis */
  flex-wrap: wrap;
  justify-content: space-between; /* main */
  align-items: center;            /* cross */
  align-content: center;          /* multi-line cross — needs wrap */
  gap: 16px;
}
```

### Items

```css
.item {
  flex-grow: 0;
  flex-shrink: 1;
  flex-basis: auto;
  flex: 0 1 auto;   /* default */
  flex: 1;          /* 1 1 0% — ignore content size when distributing */
  flex: auto;       /* 1 1 auto — respect content when distributing */
  flex: none;       /* 0 0 auto — rigid */
  align-self: center;
  order: -1;
}
```

### Patterns

```css
.navbar { display: flex; justify-content: space-between; align-items: center; }

body { display: flex; flex-direction: column; min-height: 100dvh; }
main { flex: 1; }

.card { flex: 1; display: flex; flex-direction: column; min-width: 0; }
.card-body { flex: 1; }
.card-footer { margin-top: auto; }

.item:last-child { margin-left: auto; } /* push end */
```

### Self-check

1. `flex: 1` vs `flex: auto` — when does content size still dominate?
2. Why `min-width: 0` on flex children with ellipsis?
3. How do you pin a footer to the bottom with flex?

---

# CSS Grid

**Confusion:** `auto-fill` vs `auto-fit`. Items overflow tracks. Subgrid unclear. You reach for media queries when `minmax` would suffice.

**Model:** Two-dimensional track-based layout. You size **tracks**; items are placed into cells. `fr` shares **free** space after intrinsic/fixed contributions. `auto-fit` collapses empty tracks; `auto-fill` keeps them.

**Rule:** Page layout → grid. Component axis alignment → flex. Responsive cards → `repeat(auto-fit, minmax(min(100%, 300px), 1fr))`. Allow shrink with `minmax(0, 1fr)` when needed.

### Tracks & alignment

```css
.grid {
  display: grid;
  grid-template-columns: 200px 1fr 200px;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  grid-template-columns: repeat(auto-fit, minmax(min(100%, 250px), 1fr));
  grid-auto-rows: 150px;
  grid-auto-flow: dense;
  gap: 16px;
  place-items: center;      /* align items in cells */
  justify-content: center;  /* align grid as a whole if extra space */
}
```

### Areas & placement

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
.header { grid-area: header; }

.item {
  grid-column: 1 / 3;       /* lines */
  grid-column: span 2;
  grid-area: 1 / 1 / 3 / 3; /* r-start / c-start / r-end / c-end */
}
```

### auto-fill vs auto-fit

```css
repeat(auto-fill, minmax(200px, 1fr)); /* keeps empty tracks */
repeat(auto-fit, minmax(200px, 1fr));  /* collapses empty — items stretch */
```

### Subgrid

```css
.child {
  grid-column: span 2;
  display: grid;
  grid-template-columns: subgrid; /* share parent column tracks */
}
```

Use when nested items must align to the **parent’s** tracks (card titles across a row).

### Full-bleed pattern

```css
.content {
  display: grid;
  grid-template-columns: 1fr min(65ch, 100%) 1fr;
}
.content > * { grid-column: 2; }
.full-bleed { grid-column: 1 / -1; }
```

### Flex vs Grid

```
Flex — one axis, content-driven, nav, toolbars, centering
Grid — two axes, layout-driven, pages, card grids, dashboards
```

### Self-check

1. When do you pick `auto-fit` over `auto-fill`?
2. Why might `1fr` tracks refuse to shrink?
3. What problem does subgrid solve that nested grids don’t?

---

# Responsive Design

**Confusion:** Too many breakpoints. Components break in a sidebar but not on “mobile.” Hover styles fire on touch.

**Model:** Responsive layout is **available size + input modality + user preference**. Prefer fluid sizing and **container queries** for components; media queries for viewport/page and preferences.

**Rule:** Mobile-first `min-width` queries. Components → `@container`. Prefer `clamp` over breakpoint font jumps. Feature-detect hover/pointer.

```css
/* Page / device */
@media (width <= 768px) { }
@media (768px <= width <= 1024px) { }
@media (hover: hover) and (pointer: fine) { .card:hover { } }
@media (prefers-color-scheme: dark) { }
@media (prefers-reduced-motion: reduce) { }

/* Mobile-first */
.card { padding: 16px; }
@media (min-width: 768px) { .card { padding: 24px; display: flex; } }

/* Component-owned */
.card-container {
  container-type: inline-size;
  container-name: card;
}
@container card (min-width: 400px) {
  .card { display: flex; gap: 16px; }
}

h1 { font-size: clamp(1.5rem, 4vw, 3rem); }
```

### Self-check

1. When is a container query better than a media query?
2. Why gate hover styles with `(hover: hover)`?
3. Write a fluid width: min 300, preferred 90vw, max 1200.

---

# Units

**Confusion:** `100vh` crops on mobile. `em` compounds. `rem` vs `px` arguments in reviews.

**Model:** Absolute units don’t scale. `%`/`em`/`rem` resolve against different references. Viewport units resolve against the **viewport**; `dvh`/`svh`/`lvh` disambiguate mobile browser chrome.

**Rule:** `rem` for type/spacing scale; `em` for component-local scaling; `px` for hairline borders/shadows; `ch` for measure; `dvh` for full-height UI on mobile.

```css
font-size: 1.5rem;   /* root (usually 16px → 24px) */
font-size: 1.5em;    /* relative to element’s own font-size (for font-size: parent) */
max-width: 65ch;     /* readable line length */

height: 100vh;       /* fallback */
height: 100dvh;      /* updates as browser chrome shows/hides */
height: 100svh;      /* smallest (chrome visible) */
height: 100lvh;      /* largest (chrome hidden) */
```

| Unit | Use |
|------|-----|
| `px` | borders, shadows, media-query thresholds |
| `rem` | type, spacing, component widths that scale with root |
| `em` | padding that should scale with the component’s font |
| `%` | fluid width vs parent |
| `dvh` | full-height sections on mobile |
| `ch` | max-width for prose |
| `fr` | grid free space |

### Self-check

1. Why is `100vh` risky on iOS Safari?
2. When is `em` better than `rem` for button padding?
3. What does `65ch` approximate?

---

# Logical Properties

**Confusion:** `margin-left` breaks in RTL. You write four physical inset lines for a full overlay.

**Model:** Physical directions (top/right/…) are tied to the screen. **Logical** properties follow writing mode: **block** = stacking axis, **inline** = text axis. In horizontal-tb LTR, block ≈ vertical, inline ≈ horizontal.

**Rule:** Prefer logical properties for spacing, sizing, and insets — especially shared/UI library CSS. Even in LTR-only apps, shorthands like `margin-inline` and `inset` are clearer.

```css
margin-top     → margin-block-start
margin-left    → margin-inline-start
width          → inline-size
height         → block-size
left           → inset-inline-start
top/right/bottom/left: 0 → inset: 0
text-align: left → text-align: start

.container {
  max-inline-size: 1200px;
  margin-inline: auto;
}
.card {
  padding-block: 16px;
  padding-inline: 24px;
  border-inline-start: 4px solid blue;
}
```

### Self-check

1. In RTL, what physical side is `margin-inline-start`?
2. Replace `top/right/bottom/left: 0` with one property.
3. Why prefer `text-align: start` over `left`?

---

# Aspect Ratio

**Confusion:** Old `padding-top: 56.25%` hacks linger. Images blow past the intended ratio. Video embeds collapse before load.

**Model:** `aspect-ratio` prefers a width/height relationship while sizing. If content’s intrinsic size conflicts, the box may grow — ratio is a preference, not a hard crop unless overflow clips or the replaced element is constrained (`object-fit`).

**Rule:** Set `aspect-ratio` + one size dimension (`width: 100%`). For media, pair with `object-fit` and `overflow: hidden` when you need a hard crop. Use `width`/`height` HTML attributes too to limit CLS.

```css
.video { aspect-ratio: 16 / 9; width: 100%; }
.square { aspect-ratio: 1; width: 200px; }

.thumb {
  aspect-ratio: 1;
  width: 100%;
  overflow: hidden;
}
.thumb img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
```

| Ratio | Use |
|-------|-----|
| 1 / 1 | avatars, icons |
| 16 / 9 | video, heroes |
| 4 / 3 | classic photo |
| 21 / 9 | ultra-wide banners |

### Self-check

1. Why might a box ignore `aspect-ratio: 16/9`?
2. Difference between `object-fit: cover` and `contain`?
3. How does `aspect-ratio` help CLS vs padding-bottom hacks?

---

# Colors and Custom Properties

**Confusion:** Hex themes are hard to derive. Dark mode duplicates everything. Sass variables can’t theme at runtime.

**Model:** CSS custom properties are **inherited runtime values** on the cascade. Color spaces like **OKLCH** are perceptually uniform — better for generating ramps. `color-scheme` tells the browser to restyle native UI; `accent-color` tints form controls.

**Rule:** Tokens on `:root` / `[data-theme]`. Prefer OKLCH/HSL channels for derived colors. Use `@property` when you need typed/animatable variables. Remember: an **invalid** custom property at computed-value time invalidates the declaration (fallback matters).

```css
:root {
  --color-primary: oklch(60% 0.14 250);
  --bg: white;
  --text: black;
  color-scheme: light dark;
  accent-color: var(--color-primary);
}

@media (prefers-color-scheme: dark) {
  :root { --bg: #1a1a2e; --text: #e0e0e0; }
}

.card {
  background: var(--bg);
  color: var(--text);
  margin: var(--margin, 20px); /* fallback */
}

/* Typed custom property — animatable, validated */
@property --hue {
  syntax: "<number>";
  inherits: true;
  initial-value: 200;
}
```

**Invalidation gotcha:** if `--pad` is set to `red` somehow, `padding: var(--pad)` fails and falls through — use fallbacks for defensive tokens.

```javascript
document.documentElement.style.setProperty('--color-primary', '#ff0000');
```

| CSS variables | Sass variables |
|---------------|----------------|
| Runtime, inherited | Compile-time |
| Themable via JS/DOM | Not runtime |
| No build step | Need build |

### Self-check

1. Why can CSS variables theme dark mode without a rebuild?
2. What does `color-scheme: light dark` change besides your tokens?
3. What happens if `var(--x)` is invalid and has no fallback?

---

# Transitions and Animations

**Confusion:** Animating `width` feels janky. `display: none` won’t fade. `will-change` everywhere “for performance.”

**Model:** **Composite** properties (`transform`, `opacity`) can animate on the GPU without layout. Layout properties trigger **reflow**. Discrete properties (`display`) need special handling (`@starting-style` / `allow-discrete` — see Modern). Respect `prefers-reduced-motion`.

**Rule:** Animate `transform`/`opacity`. Transition specific properties, not `all`. Apply `will-change` temporarily, never globally.

```css
.button {
  transition: background 0.3s ease, transform 0.2s ease;
}
.button:hover { transform: scale(1.05); }

/* Can’t transition: display, font-family, position, content */
.modal {
  visibility: hidden;
  opacity: 0;
  transition: opacity 0.3s, visibility 0.3s;
}
.modal.active { visibility: visible; opacity: 1; }

@keyframes slideIn {
  from { transform: translateX(-100%); opacity: 0; }
  to   { transform: translateX(0); opacity: 1; }
}
.element {
  animation: slideIn 0.5s ease-out forwards;
}

/* CHEAP: transform, opacity */
/* EXPENSIVE: width, height, top, left, margin */

@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

### Self-check

1. Why is `transform` preferred over animating `top`?
2. What’s wrong with `* { will-change: transform; }`?
3. How do you honor reduced motion?

---

# Pseudo-classes and Pseudo-elements

**Confusion:** `:focus` outlines on every mouse click. `:nth-child` vs `:nth-of-type`. Pseudo-elements used where extra DOM would be clearer.

**Model:** **Pseudo-classes** select elements in a state (`:hover`, `:invalid`). **Pseudo-elements** style generated fragments (`::before`, `::selection`). Structural pseudos count tree position.

**Rule:** Style keyboard focus with `:focus-visible`. Prefer real DOM for complex UI chrome; use `::before/::after` for light decoration.

```css
button:focus { outline: none; }
button:focus-visible { outline: 2px solid #4A90D9; outline-offset: 2px; }

input:required:invalid { }
li:nth-child(3n+1) { }
li:nth-child(-n+5) { }              /* first 5 */
li:nth-child(n+3):nth-child(-n+7) { } /* 3–7 */

.quote::before { content: "\201C"; }
::selection { background: #4A90D9; color: white; }
li::marker { color: #4A90D9; }

input[type="checkbox"] {
  appearance: none;
  width: 20px; height: 20px;
  border: 2px solid #ccc;
}
input[type="checkbox"]:checked::after { content: "✓"; }
```

### Self-check

1. `:focus` vs `:focus-visible` — what changes for mouse users?
2. Does `::before` work without `content`?
3. `:nth-child(odd)` vs `:nth-of-type(odd)` on mixed tags?

---

# Typography

**Confusion:** FOIT/FOUT flashes. Unitless vs unit `line-height` inheritance bugs. Multi-line clamp feels hacky. Font swap shifts layout.

**Model:** Text layout is font metrics + line boxes + wrapping rules. `font-display` trades blank text vs fallback flash vs layout shift. Unitless `line-height` inherits as a **multiplier**; unit values inherit as absolute computed lengths.

**Rule:** `font-display: swap` for body branding; `optional` when CLS matters more than custom face. Unitless line-height. Set explicit `size-adjust` / matching fallback metrics when polishing.

```css
body {
  font-family: system-ui, 'Segoe UI', Roboto, sans-serif;
  line-height: 1.5; /* unitless — inherits multiplier */
}
h1 { line-height: 1.2; }

@font-face {
  font-family: 'MyFont';
  src: url('/fonts/myfont.woff2') format('woff2');
  font-weight: 400;
  font-display: swap; /* FOUT: fallback first, then swap */
}

/* optional — almost no late swap; best CLS, may never show custom font */
```

```
auto     — browser decides
block    — invisible up to ~3s (FOIT)
swap     — fallback immediately (FOUT) ← body default choice
optional — use custom only if already available
```

```css
.truncate {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
.truncate-multi {
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 3;
  overflow: hidden;
}
.long-text { overflow-wrap: break-word; hyphens: auto; }
.nums { font-variant-numeric: tabular-nums; }
```

### Self-check

1. Why prefer unitless `line-height`?
2. `swap` vs `optional` — which prioritizes CLS?
3. What three properties make single-line ellipsis work?

---

# Overflow and Scrolling

**Confusion:** `overflow: hidden` creates a scroll container and breaks sticky. Scroll chaining pulls the page behind a modal. Custom scrollbars differ by engine.

**Model:** Overflow decides clipping and whether a box is a **scroll container**. `clip` clips without becoming a scroll container. Scroll snap / overscroll control user scroll UX. Sticky is sensitive to overflow ancestors (see Positioning).

**Rule:** Modals: `overscroll-behavior: contain` on the scrollable panel. Prefer `clip` when you only need clip, not scroll. Always pair smooth scroll with reduced-motion.

```css
overflow: visible; /* default */
overflow: hidden;  /* clip + scroll container (JS can still scroll) */
overflow: clip;    /* clip only — no scroll container */
overflow: auto;    /* scrollbar if needed */
overflow: scroll;  /* always scrollports */

html { scroll-behavior: smooth; }
@media (prefers-reduced-motion: reduce) {
  html { scroll-behavior: auto; }
}

.carousel {
  display: flex;
  overflow-x: auto;
  scroll-snap-type: x mandatory;
}
.carousel-item {
  scroll-snap-align: start;
  flex: 0 0 100%;
}

.modal-body { overflow-y: auto; overscroll-behavior: contain; }
body { overscroll-behavior-y: none; } /* reduce pull-to-refresh */

.container {
  scrollbar-width: thin;
  scrollbar-color: #888 #f0f0f0;
}
```

### Self-check

1. `hidden` vs `clip` — which can break `position: sticky` and why?
2. What does `overscroll-behavior: contain` prevent?
3. Why disable smooth scrolling under reduced motion?

---

# CSS Functions

**Confusion:** `calc` breaks without spaces. `clamp` vs media queries. Safe areas ignored on notched phones.

**Model:** Functions compute values at used-value time from other values (`calc`, `min`/`max`/`clamp`), tokens (`var`), environment (`env`), or colors (`color-mix`).

**Rule:** Spaces around `+`/`-` in `calc`. Prefer `clamp` for fluid type/space. Always fallback `env(safe-area-inset-*)`.

```css
width: calc(100% - 250px);          /* spaces required around - */
width: min(90vw, 1200px);
width: max(300px, 50vw);
width: clamp(300px, 90vw, 1200px);  /* min, preferred, max */
padding: env(safe-area-inset-top, 20px);

background: color-mix(in oklch, var(--primary), black 20%);
background: linear-gradient(135deg, #f00, #00f);
filter: blur(4px) brightness(1.2);
backdrop-filter: blur(10px);

.gradient-text {
  background: linear-gradient(135deg, #667eea, #764ba2);
  background-clip: text;
  -webkit-background-clip: text;
  color: transparent;
}
```

### Self-check

1. Why is `calc(100%-250px)` invalid?
2. Express “never wider than 1200, never narrower than 300, else 90vw.”
3. What is `env(safe-area-inset-bottom)` for?

---

# Modern CSS Features

**Confusion:** Specificity wars with third-party CSS. Nesting without a preprocessor. Entering from `display: none` won’t animate. View transitions feel magical/opaque.

**Model:** Modern CSS adds **cascade control** (`@layer`, `@scope`), **author ergonomics** (nesting), and **lifecycle hooks** (`@starting-style`, view transitions). Feature-detect with `@supports`.

**Rule:** Put resets/vendor in layers. Scope component CSS when global leakage hurts. Use `@starting-style` for entry animations from `display: none`. Progressive-enhance view transitions.

```css
@layer reset, base, components, utilities;

.card {
  padding: 16px;
  .title { font-size: 1.5rem; }
  &:hover { box-shadow: 0 2px 8px rgb(0 0 0 / 10%); }
  @media (max-width: 768px) { padding: 8px; }
}

@scope (.card) to (.card-footer) {
  h2 { font-size: 1.5rem; }
}

.modal {
  display: none;
  opacity: 0;
  transition: opacity 0.3s, display 0.3s allow-discrete;
  @starting-style { opacity: 0; }
}
.modal.open { display: block; opacity: 1; }

@supports selector(:has(*)) {
  .card:has(img) { padding: 0; }
}

::view-transition-old(root) { animation: fade-out 0.3s; }
::view-transition-new(root) { animation: fade-in 0.3s; }
```

### Self-check

1. Do unlayered styles beat layered styles regardless of specificity?
2. What problem does `@starting-style` solve?
3. When would you reach for `@scope` instead of a CSS Module?

---

# Performance

**Confusion:** “CSS isn’t a performance issue.” Animating layout. Reading `offsetHeight` in a write loop. Giant stylesheets for below-the-fold content.

**Model:** Cost ladder: **Reflow (layout) → Repaint → Composite**. Reading geometry forces layout; writing then reading in a loop = **layout thrashing**. Containment tells the browser what won’t affect the outside.

**Rule:** Animate transform/opacity. Batch DOM reads then writes. Use `content-visibility: auto` for long pages. Inline critical CSS for first paint.

```
Reflow:  width, height, margin, padding, font-size, offsetHeight reads
Repaint: color, background, visibility, box-shadow
Composite: transform, opacity
```

```javascript
// BAD — thrash
for (const el of elements) {
  el.style.width = container.offsetWidth + 'px';
}
// GOOD — read once, write many
const width = container.offsetWidth;
for (const el of elements) {
  el.style.width = width + 'px';
}
```

```css
.card { contain: content; }
.section {
  content-visibility: auto;
  contain-intrinsic-size: 0 500px;
}
```

```
Animate transform/opacity only
contain: content for card islands
content-visibility: auto for long feeds
will-change sparingly and temporarily
Batch reads, then writes
Inline critical CSS
```

### Self-check

1. Rank `opacity`, `left`, `background-color` from cheapest to costliest to animate.
2. What is layout thrashing?
3. What does `content-visibility: auto` skip?

---

# Architecture

**Confusion:** “Just use Tailwind” vs “BEM forever.” Specificity wars with third-party CSS. Tokens live in three places.

**Model:** Architecture is **ownership + collision control + change velocity**. Naming (BEM), isolation (Modules/Shadow), utilities (Tailwind), and layers (`@layer`) are tools for those goals — not religions.

**Rule:** Pick a default for the team, define escape hatches, put design tokens in CSS variables, and use `@layer` at integration boundaries. Explain tradeoffs in interviews, don’t recite a winner.

### BEM (collision control via naming)

```css
.card { }
.card__title { }
.card--featured { }
```

Every selector stays ~0-1-0. Good for large multi-team plain CSS.

### Isolation approaches

```html
<!-- Tailwind — utility velocity; watch bundle + design consistency -->
<article class="p-4 border rounded-lg">...</article>
```

```css
/* CSS Modules — local by default, no runtime */
.card { padding: 16px; }
```

```js
import styles from './Card.module.css';
```

### Ownership with layers

```css
@layer reset, tokens, vendor, components, utilities, overrides;
```

Put third-party in `vendor`. Keep app components in `components`. Utilities last among layered; unlayered only for true escapes.

### Token rule

One source of truth: CSS variables (optionally generated from JSON). Components consume tokens, not raw hex — themes and JS become trivial.

### Interview framing

```
BEM           — explicit ownership in global CSS, long-lived large teams
Modules       — component apps, compile-time isolation, low drama
Tailwind      — speed + constraints; needs lint/design discipline
CSS-in-JS     — dynamic theming; cost is runtime/tooling complexity
@layer        — integrate systems without !important
Shadow DOM    — true style encapsulation (design systems / widgets)
```

### Self-check

1. What problem does BEM actually solve (be precise)?
2. When do CSS Modules beat BEM for the same React app?
3. Where should third-party CSS sit in an `@layer` stack and why?

---

## Quick revisit map

| Before an interview | Re-say the model in one sentence |
|---------------------|----------------------------------|
| Layout bugs | Formatting context + containing block |
| Won’t shrink / truncate | Intrinsic min-size / `min-width: 0` |
| Override fights | Cascade: layer → specificity → order + keywords |
| Janky UI | Reflow vs composite |
| Structure debate | Ownership + collision + tokens |
