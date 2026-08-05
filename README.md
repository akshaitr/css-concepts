## Table of Contents

### Foundations

1. [Box Model](#box-model)
2. [Cascade, Specificity & Keywords](#cascade-specificity--keywords)
3. [Formatting Contexts](#formatting-contexts)
4. [Containing Block, Positioning & Stacking](#containing-block-positioning--stacking)
5. [Intrinsic Sizing](#intrinsic-sizing)

### Layout systems

6. [Display essentials](#display-essentials)
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

Every element is a rectangle: content → padding → border → margin. `width`/`height` mean different things under `content-box` vs `border-box`. Vertical margins of adjoining blocks **collapse**; horizontal never do.

```
┌──────────────────────────────────────────┐
│                 MARGIN                   │
│  ┌────────────────────────────────────┐  │
│  │              BORDER                │  │
│  │  ┌──────────────────────────────┐  │  │
│  │  │           PADDING            │  │  │
│  │  │  ┌────────────────────────┐  │  │  │
│  │  │  │        CONTENT         │  │  │  │
│  │  │  └────────────────────────┘  │  │  │
│  │  └──────────────────────────────┘  │  │
│  └────────────────────────────────────┘  │
└──────────────────────────────────────────┘
```

### Trap: width doesn’t mean “total size”

```css
/* content-box (default) — padding/border ADD to width */
.box {
  box-sizing: content-box;
  width: 200px;
  padding: 20px;
  border: 5px solid black;
  /* Total: 250px */
}

/* border-box — width INCLUDES padding + border */
.box {
  box-sizing: border-box;
  width: 200px;
  padding: 20px;
  border: 5px solid black;
  /* Total: 200px */
}

*,
*::before,
*::after {
  box-sizing: border-box;
}
```

### Trap: margin collapsing

When two vertical margins touch, the **larger** wins — they do not add.

```css
.box1 {
  margin-bottom: 30px;
}
.box2 {
  margin-top: 20px;
}
/* Gap: 30px, not 50px */
```

**Collapses:** adjacent siblings; parent ↔ first/last child (nothing between); empty block’s own top+bottom.

**Stops collapse:** parent padding/border; `overflow` ≠ `visible`; flex/grid container; `display: flow-root`.

```css
.parent {
  display: flow-root;
} /* child margins stay inside */
```

### Outline vs border

```css
.box {
  border: 2px solid red;
} /* affects layout */
.box {
  outline: 2px solid blue;
  outline-offset: 4px;
} /* does not */

/* BAD */
*:focus {
  outline: none;
}
/* GOOD */
*:focus-visible {
  outline: 2px solid #4a90d9;
  outline-offset: 2px;
}
```

### Self-check

1. Why does `width: 100%` + `padding: 16px` overflow under `content-box`?
2. Name three ways to stop parent/child margin collapse.
3. Why prefer `outline` over `border` for focus rings?

---

# Cascade, Specificity & Keywords

Conflict resolution order:

```
1. Origin + importance (!important flips user/author ranking)
2. @layer order (earlier layers lose; unlayered beats all layers)
3. Specificity
4. Source order (last wins)
```

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
!important (by origin) → strongest lever (avoid in app CSS)
Inline → ID → class/attr/:pseudo → element/::pseudo → * / :where()
```

### Worked conflict: layer vs specificity

```css
@layer components {
  #widget .button {
    color: blue;
  } /* specificity 1-1-0, but layered */
}
.button {
  color: red;
} /* 0-1-0, unlayered — WINS */
```

Unlayered author styles beat layered ones **regardless of specificity**. That is the point of `@layer` for vendor CSS.

### Worked conflict: unset vs revert

```css
/* Inherited property */
.card {
  color: navy;
}
.card .muted {
  color: unset;
} /* → inherits navy (like inherit) */

/* Non-inherited property */
.button {
  display: flex;
}
.button.reset {
  display: unset;
} /* → initial (inline for span, etc.) — NOT “browser button” */
.button.reset {
  display: revert;
} /* → user-agent button display */
```

| Keyword        | Means                                   | Typical use              |
| -------------- | --------------------------------------- | ------------------------ |
| `inherit`      | Parent’s computed value                 | Force inheritance        |
| `initial`      | Spec initial                            | Rare; aggressive         |
| `unset`        | Inherit if inherited prop, else initial | Soft one-property reset  |
| `revert`       | Roll toward UA/user style               | Unstyle an island        |
| `revert-layer` | Previous `@layer` only                  | Escape utilities cleanly |

```css
@layer reset, base, components, utilities;
@layer components {
  .nav a {
    color: white;
  }
}
.prose a {
  color: revert-layer;
}
```

### Selectors worth knowing

```css
.card > p {
}
h2 + p {
}
h2 ~ p {
}
[href^='https'] {
}
[href$='.pdf'] {
}
[data-active] {
}
:is(#header, .nav) a {
} /* max specificity of args */
:where(.card, .panel) h2 {
} /* 0 specificity for :where */
.card:has(img) {
}
```

### Escaping !important

```css
:where(.button) {
  color: blue;
} /* zero-specificity base */
.button {
  color: red;
}
@layer vendor {
  .button {
    color: blue;
  }
}
.button {
  color: red;
} /* unlayered wins */
```

### Self-check

1. Why does an unlayered `.button` beat `#x .button` inside `@layer components`?
2. `unset` vs `revert` on `display` — what differs?
3. What specificity does `:where(.a, .b) p` contribute from `:where`?

---

# Formatting Contexts

A **formatting context** is a layout arena. Block layout uses a **BFC**. Flex and Grid create their own. Inside a new BFC: floats are contained, margins don’t collapse through the boundary, and layout ignores outside floats.

**Prefer intentional BFCs** (`flow-root`, flex, grid) over cargo-cult `overflow: hidden`.

```css
.el {
  display: flow-root; /* purpose-built */
  display: flex;
  display: grid;
  overflow: auto; /* any value except visible also creates a BFC */
  contain: layout;
}
```

```css
.clearfix {
  display: flow-root;
} /* contain floats */
.card {
  display: flow-root;
} /* stop margin leak */
.beside-float {
  display: flow-root;
} /* sit beside a float cleanly */
```

Floats leave normal flow but still affect line boxes. A BFC parent expands to enclose its floats — that is float containment.

**Flex/Grid note:** margins between flex/grid _items_ do not collapse. Item margins also don’t collapse through the container the way block children can.

### Self-check

1. Why does `flow-root` fix margin leak without clipping?
2. Name two BFC triggers that aren’t `overflow`.

---

# Containing Block, Positioning & Stacking

Three separate models. Trace them in order: **containing block → position mode → stacking context**.

---

## 1. Containing block

Offsets (`top`, `left`, `inset`, percentages for absolutely positioned boxes, etc.) resolve against the **containing block** — not always “nearest `position: relative` ancestor.”

| For `position: absolute`                                                                                          | Containing block is…                        |
| ----------------------------------------------------------------------------------------------------------------- | ------------------------------------------- |
| Ancestor with `position` ≠ `static`                                                                               | That ancestor’s padding edge                |
| Ancestor with `transform`, `filter`, `perspective`, `will-change: transform`, `contain: paint`, `backdrop-filter` | That ancestor — even if `position: static`  |
| None of the above                                                                                                 | Initial containing block (viewport-related) |

**`position: fixed`:** normally the viewport. **Same transform/filter/perspective ancestors** make fixed behave like absolute inside that ancestor.

### Worked bug: “fixed navbar scrolls away”

```html
<div class="app" style="transform: translateZ(0)">
  <!-- GPU hint / animation remnant -->
  <header style="position: fixed; top: 0">Nav</header>
</div>
```

`transform` on `.app` creates a containing block. `fixed` is now stuck to `.app`, not the viewport — it scrolls away with the app.

**Fix:** don’t put `transform`/`filter` on an ancestor of fixed UI; move the fixed node outside, or use a different compositing approach.

### Worked bug: absolute sticks to the wrong box

```html
<section class="card">
  <!-- position: static, but… -->
  <div class="media" style="filter: blur(0)">
    <!-- filter creates CB -->
    <img />
    <span class="badge" style="position: absolute; top: 0; right: 0">New</span>
  </div>
</section>
```

You intended `.card { position: relative }`, but the badge positions against `.media` because `filter` created a containing block first.

---

## 2. Position modes

```css
/* static   — default; top/left/z-index ignored */
/* relative — offset from normal position; original space kept; CB for abs kids */
/* absolute — out of flow; positioned vs containing block above */
/* fixed    — out of flow; vs viewport unless CB-creating ancestor */
/* sticky   — relative until threshold, then stuck within ancestor */
```

```css
.parent {
  position: relative;
}
.badge {
  position: absolute;
  top: -8px;
  right: -8px;
}
```

### Worked bug: sticky does nothing

Sticky fails unless **all** of these hold:

1. A threshold: `top`, `bottom`, `left`, or `right` (not auto)
2. A scrolling ancestor (the sticky element’s nearest scrollport)
3. No ancestor between sticky and that scrollport with `overflow: hidden | auto | scroll` (unless that ancestor _is_ the intended scrollport)
4. Room to stick — parent taller than the sticky element; sticky can’t escape its parent

```css
/* Broken: overflow on a wrapper clips the sticky chain */
.sidebar {
  overflow: auto;
}
.sidebar h2 {
  position: sticky;
  top: 0;
} /* sticks inside sidebar — OK if that’s intended */

/* Broken: overflow:hidden on a decorative parent of a page-level sticky header */
.page-wrap {
  overflow-x: hidden;
} /* often kills sticky headers */
.site-header {
  position: sticky;
  top: 0;
}
```

**Fix:** move overflow to a lower child, use `overflow-x: clip` where you only need clipping (see Overflow), or stick within the scroll container you actually control.

---

## 3. Stacking contexts

`z-index` only competes **inside the same stacking context**. Nested contexts are ordered by their parent contexts first.

**Common creators:** `position` + `z-index` ≠ `auto`; `opacity < 1`; `transform`; `filter`; `perspective`; `clip-path`; `isolation: isolate`; `will-change`; `contain: paint`.

### Worked bug: z-index 9999 still behind

```css
.modal-root {
  position: relative;
  z-index: 1;
}
.dropdown {
  position: relative;
  z-index: 2;
} /* sibling, later painting world */
.modal-root .dialog {
  position: relative;
  z-index: 9999;
}
/* .dialog loses to .dropdown — parent context is only z-index: 1 */
```

**Fix:** raise `.modal-root`’s context, render the modal in a portal at the top of the DOM, or use the **top layer** (`dialog` / `popover`). For local UI, `isolation: isolate` creates a clean stacking root without opacity/transform side effects:

```css
.component {
  isolation: isolate;
}
```

### Centering cheatsheet

```css
.parent {
  display: flex;
  justify-content: center;
  align-items: center;
}
.parent {
  display: grid;
  place-items: center;
}
.child {
  position: absolute;
  inset: 50% auto auto 50%;
  transform: translate(-50%, -50%);
}
.child {
  width: 500px;
  margin-inline: auto;
}
```

### Self-check

1. List three CSS properties that create a containing block for fixed descendants.
2. Why can `overflow-x: hidden` on `body` break a sticky header?
3. Explain why child `z-index: 9999` can still paint under a sibling of its parent.

---

# Intrinsic Sizing

Boxes have preferred and minimum content sizes. Flex/grid’s **automatic minimum size** often floors at roughly `min-content`, so items refuse to shrink below the longest word/image until you override.

### Keywords

```css
width: min-content; /* shrink-wrap to longest unbreakable piece */
width: max-content; /* fit content without wrapping */
width: fit-content; /* ~min(max-content, available) */
width: fit-content(200px);
```

### Before / after: truncation in a flex row

```css
/* BEFORE — text won’t ellipsis; item won’t shrink */
.row {
  display: flex;
}
.title {
  flex: 1;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
/* min-width: auto ≈ min-content → floor is full text width */

/* AFTER */
.title {
  flex: 1;
  min-width: 0; /* allow shrink below min-content */
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
```

### Before / after: equal grid columns overflowing

```css
/* BEFORE — 1fr is often minmax(auto, 1fr); auto won’t go below min-content */
.grid {
  grid-template-columns: 1fr 1fr;
}

/* AFTER */
.grid {
  grid-template-columns: minmax(0, 1fr) minmax(0, 1fr);
}
```

| Value            | Role                                       |
| ---------------- | ------------------------------------------ |
| `auto`           | Size to content (intrinsic)                |
| `1fr`            | Share free space; min often still `auto`   |
| `%`              | Percentage of container / CB               |
| `minmax(0, 1fr)` | Flexible **and** allowed to shrink to zero |

Use `width: min-content` on purpose for shrink-wrapped chips/tags; use `max-content` when you must avoid wrapping (careful on small screens).

### Self-check

1. Why doesn’t `flex: 1` alone enable ellipsis?
2. `1fr` vs `minmax(0, 1fr)` — what’s the practical difference?

---

# Display essentials

`display` sets **outer** role (how the box participates among siblings) and **inner** layout mode (how children layout). Real UI layout → flex/grid (next sections). This is the compact leftover.

| Value                | Behavior                                                               |
| -------------------- | ---------------------------------------------------------------------- |
| `block`              | New line, full width by default, full box model                        |
| `inline`             | Shares line; ignores `width`/`height`; vertical margin doesn’t push    |
| `inline-block`       | Inline placement + respects width/height/margin                        |
| `contents`           | Box disappears; children join grandparent’s layout                     |
| `flow-root`          | New BFC (see Formatting Contexts)                                      |
| `none`               | Removed from layout + a11y tree; children can’t override               |
| `visibility: hidden` | Invisible, space kept; children can set `visible`                      |
| `opacity: 0`         | Invisible, space kept, **still interactive + accessible** — fades only |

```css
.badge {
  display: inline-block;
  width: 100px;
  height: 30px;
}
.wrapper {
  display: contents;
}
```

---

# Flexbox

One-dimensional layout: distribute space along a **main axis**, align on the **cross axis**.

```
flex-direction: row (default)
─────────────────────────────────────────
main → → → → → → → → → → → → → → → → →
│  [item]  [item]  [item]               │
│         cross (↓)                     │
─────────────────────────────────────────

flex-direction: column
main ↓        cross →
[item]
[item]
[item]
```

`flex-wrap: wrap` allows multiple **flex lines**. Then `align-content` distributes those lines on the cross axis; `align-items` still aligns items within each line.

---

## Container: axes and alignment

```css
.container {
  display: flex;
  flex-direction: row; /* row | row-reverse | column | column-reverse */
  flex-wrap: nowrap; /* nowrap | wrap | wrap-reverse */
  flex-flow: row wrap; /* shorthand: direction + wrap */

  justify-content: flex-start; /* MAIN axis: pack / space items */
  align-items: stretch; /* CROSS axis: default stretch */
  align-content: stretch; /* CROSS axis, multi-line only — needs wrap */

  gap: 16px; /* preferred over margin hacks between items */
  row-gap: 16px;
  column-gap: 8px;
}
```

### Which alignment property?

| Goal                                          | Property          |
| --------------------------------------------- | ----------------- |
| Space along main axis (horizontal in a row)   | `justify-content` |
| Align items on cross axis (vertical in a row) | `align-items`     |
| One item overrides cross alignment            | `align-self`      |
| Distribute **lines** when wrapped             | `align-content`   |
| Space between items (both axes)               | `gap`             |

```css
justify-content: flex-start | flex-end | center | space-between | space-around |
  space-evenly;
align-items: stretch | flex-start | flex-end | center | baseline;
```

**Gotcha:** `align-content` does nothing if there’s only one flex line (`nowrap` or not enough items to wrap).

---

## Item sizing algorithm (the part that builds understanding)

For each flex item, think in three steps on the **main** size:

### Step 1 — `flex-basis` (starting main size)

```css
flex-basis: auto; /* use width/height if set, else content size */
flex-basis: 0; /* start from zero — content ignored for distribution */
flex-basis: 200px; /* explicit start */
```

### Step 2 — `flex-shrink` if overflow

If the sum of basis sizes exceeds the container, items shrink proportional to `flex-shrink` (default `1`). An item with `flex-shrink: 0` stays rigid.

**Floor:** automatic minimum size ≈ `min-content` unless you set `min-width: 0` / `min-height: 0` (see [Intrinsic Sizing](#intrinsic-sizing)).

### Step 3 — `flex-grow` if free space remains

Extra space is distributed proportional to `flex-grow` (default `0`).

### The `flex` shorthand

```css
flex: 0 1 auto; /* default — don’t grow, may shrink, basis auto */
flex: 1; /* 1 1 0% — grow/shrink equally from zero basis */
flex: auto; /* 1 1 auto — grow/shrink but start from content size */
flex: none; /* 0 0 auto — rigid */
flex: 1 0 auto; /* grow, never shrink */
```

**Why equal columns fail with `flex: auto`:** each item’s content size differs, so they start unequal and then grow equally — taller content stays wider. Prefer `flex: 1` (basis 0) or `flex: 1 1 0` plus `min-width: 0` for equal flexible columns.

**Numeric intuition:**

```css
.a {
  flex-grow: 1;
}
.b {
  flex-grow: 2;
}
/* With basis 0, B gets ~2× the free space of A */
```

---

## Common patterns (with why)

```css
/* Navbar: group left / right */
.navbar {
  display: flex;
  align-items: center;
  gap: 16px;
}
.navbar .spacer {
  flex: 1;
} /* OR */
.navbar .actions {
  margin-left: auto;
} /* push trailing cluster */

/* Sticky footer */
body {
  display: flex;
  flex-direction: column;
  min-height: 100dvh;
}
main {
  flex: 1;
} /* absorbs free vertical space */

/* Card: body fills, footer pinned */
.card {
  display: flex;
  flex-direction: column;
  min-height: 100%;
  min-width: 0;
}
.card-body {
  flex: 1;
}
.card-footer {
  margin-top: auto;
} /* push to cross-end of column flex */

/* Truncation */
.item {
  flex: 1;
  min-width: 0;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
```

---

## Extra behaviors worth knowing

```css
/* Absolute flex child — positioned against the flex container’s padding box
   (the container is the CB if it’s the containing block). Removed from flex flow. */
.container {
  display: flex;
  position: relative;
}
.container .badge {
  position: absolute;
  top: 0;
  right: 0;
}

/* order — visual only; keep DOM order for a11y/tab */
.item:nth-child(1) {
  order: 2;
}

/* Nested flex — outer for page chrome, inner for component alignment */
```

**`gap` vs margin:** `gap` does not collapse and doesn’t add space outside the container edges the way item margins can. Prefer `gap` for item spacing; use `margin-left: auto` for “push this item/group to the end.”

---

## Flex traps checklist

| Symptom                  | Likely cause                       | Fix                                                 |
| ------------------------ | ---------------------------------- | --------------------------------------------------- |
| Unequal “equal” columns  | `flex: auto` / content-sized basis | `flex: 1` + `min-width: 0`                          |
| Text won’t truncate      | `min-width: auto`                  | `min-width: 0` on the item                          |
| `align-content` ignored  | Single line                        | `flex-wrap: wrap` + enough items/width              |
| Image blows the row      | Intrinsic min size                 | `min-width: 0` on item; `img { max-width: 100% }`   |
| Vertical centering fails | Confused main/cross                | Check `flex-direction`; use `align-items` for cross |

### Self-check

1. Walk basis → shrink → grow for three items with `flex: 1`, `flex: 2`, `flex: none` in a 600px row (basis 0).
2. When does `align-content: center` visibly do nothing?
3. Why does `margin-left: auto` on one item push it to the end?
4. `flex: 1` vs `flex: auto` for a row of cards with different title lengths?

---

# CSS Grid

Two-dimensional layout: you size **tracks** (rows/columns); items occupy **cells** (and can span). Flex distributes along one axis from content; Grid lets you design the skeleton first.

```
          line 1     line 2     line 3     line 4
            |          |          |          |
 line 1  ———+——————————+——————————+——————————+———
            |  cell    |  cell    |  cell    |
 line 2  ———+——————————+——————————+——————————+———
            |  cell    |  cell    |  cell    |
 line 3  ———+——————————+——————————+——————————+———

Tracks = spaces BETWEEN lines. Placement uses line numbers or names.
```

---

## Explicit vs implicit grid

```css
.grid {
  display: grid;
  /* Explicit — you defined these */
  grid-template-columns: 200px 1fr 200px;
  grid-template-rows: auto 1fr;

  /* Implicit — created when items are placed outside explicit tracks */
  grid-auto-rows: 120px;
  grid-auto-columns: minmax(0, 1fr);
  grid-auto-flow: row; /* row | column | dense */
}
```

`dense` packs later items into earlier holes left by spanning items — visual order can diverge from DOM order (a11y caution).

---

## Track sizing (how to think, not the full spec)

1. **Fixed** tracks (`px`, definite sizes) take their space first.
2. **Intrinsic** tracks (`auto`, `min-content`, `max-content`) size to content.
3. **Flexible** tracks (`fr`) share **remaining free space**.

```css
grid-template-columns: 200px 1fr 1fr; /* fixed + free space split */
grid-template-columns: repeat(
  3,
  minmax(0, 1fr)
); /* equal flexible, can shrink */
grid-template-columns: repeat(auto-fit, minmax(min(100%, 250px), 1fr));
```

`minmax(min, max)` clamps a track. `min(100%, 250px)` inside `minmax` prevents overflow on narrow containers (the “don’t require 250px when the parent is 200px” fix).

---

## Alignment: items vs the grid

```css
.grid {
  justify-items: stretch; /* default — horizontal alignment OF EACH ITEM in its cell */
  align-items: stretch; /* vertical alignment OF EACH ITEM in its cell */
  place-items: center; /* both */

  justify-content: start; /* if total grid width < container — position the grid */
  align-content: start; /* same for block axis */
  place-content: center;
}
```

| Property                            | Aligns                                              |
| ----------------------------------- | --------------------------------------------------- |
| `justify-items` / `align-items`     | Items inside their cells                            |
| `justify-self` / `align-self`       | One item in its cell                                |
| `justify-content` / `align-content` | The whole grid when tracks don’t fill the container |

---

## Placement

```css
.item {
  grid-column: 1 / 3; /* start line / end line (end is exclusive) */
  grid-column: span 2; /* span 2 tracks from auto-placement */
  grid-row: 2 / 3;
  grid-area: 1 / 1 / 3 / 3; /* row-start / col-start / row-end / col-end */
}

/* Named lines */
.grid {
  grid-template-columns: [full-start] 1fr [content-start] minmax(
      0,
      70ch
    ) [content-end] 1fr [full-end];
}
.full {
  grid-column: full-start / full-end;
}
.content {
  grid-column: content-start / content-end;
}
```

### Template areas

```css
.layout {
  display: grid;
  grid-template-columns: 240px 1fr 240px;
  grid-template-rows: auto 1fr auto;
  grid-template-areas:
    'header  header  header'
    'sidebar content aside'
    'footer  footer  footer';
  min-height: 100dvh;
  gap: 0;
}
.header {
  grid-area: header;
}
.sidebar {
  grid-area: sidebar;
}
.content {
  grid-area: content;
}
.aside {
  grid-area: aside;
}
.footer {
  grid-area: footer;
}

@media (width <= 768px) {
  .layout {
    grid-template-columns: 1fr;
    grid-template-areas:
      'header'
      'content'
      'sidebar'
      'aside'
      'footer';
  }
}
```

Each area name must form a single rectangle.

---

## auto-fill vs auto-fit (concrete story)

Both: “as many tracks as fit given this `minmax`.”

```css
.grid {
  width: 900px;
  gap: 0;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
}
/* 900px fits four 200px tracks → 4 columns.
   With only 2 items, auto-fill KEEPS 2 empty tracks.
   Items stay in the first two columns; leftover track space remains as empty columns. */

.grid {
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
}
/* Same 2 items: empty tracks COLLAPSE.
   The 2 items stretch (1fr) across the full 900px. */
```

**Rule of thumb:** responsive card grids → `auto-fit`. Prefer reserved empty columns (dashboard skeletons) → `auto-fill`.

Always consider:

```css
repeat(auto-fit, minmax(min(100%, 250px), 1fr));
```

so a single column can shrink below 250px on tiny screens.

---

## Subgrid — problem → solution

**Problem:** a row of cards is a grid, but titles/footers inside each card don’t line up across cards — each card’s inner grid is independent.

**Solution:** child grid inherits parent tracks with `subgrid`.

```css
.cards {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 24px;
}
.card {
  display: grid;
  grid-row: span 3;
  grid-template-rows: subgrid; /* rows align with parent’s row tracks */
  gap: inherit;
}
.card h2 {
  grid-row: 1;
}
.card .body {
  grid-row: 2;
}
.card .footer {
  grid-row: 3;
}
```

Use when nested alignment must match the **parent** track lines. Browser support is mainstream now; still progressive-enhance if you support very old engines.

---

## Patterns

```css
/* Responsive cards — often zero media queries */
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(min(100%, 280px), 1fr));
  gap: 24px;
}

/* Full-bleed inside a centered measure */
.content {
  display: grid;
  grid-template-columns: 1fr min(65ch, 100%) 1fr;
}
.content > * {
  grid-column: 2;
}
.full-bleed {
  grid-column: 1 / -1;
}

/* Holy grail-ish with 1fr middle */
.app {
  display: grid;
  grid-template: auto 1fr auto / auto 1fr auto;
  min-height: 100dvh;
}
```

---

## Flex vs Grid — choosing

| Use Flex when…                         | Use Grid when…                       |
| -------------------------------------- | ------------------------------------ |
| One axis, content-driven               | Two axes, layout-driven              |
| Nav, toolbars, input groups, centering | Page shells, card grids, dashboards  |
| Unknown number of items packing a row  | Explicit rows **and** columns matter |
| Component internals                    | Page / section skeleton              |

They compose: **grid for the page, flex inside components**.

### Self-check

1. Draw line numbers for `grid-template-columns: 1fr 1fr 1fr` and place an item `grid-column: 2 / 4`.
2. Why do empty columns remain with `auto-fill` but vanish with `auto-fit`?
3. When do `justify-items` and `justify-content` differ?
4. What problem does subgrid solve that a nested independent grid doesn’t?
5. Why `minmax(0, 1fr)` instead of `1fr` for equal columns with long content?

---

# Responsive Design

Responsive layout reacts to **available size**, **input modality**, and **user preference** — not only “mobile vs desktop widths.”

### Decision guide

| Question                                                                   | Prefer                                           |
| -------------------------------------------------------------------------- | ------------------------------------------------ |
| Does this **component** change when its **slot** is narrow (e.g. sidebar)? | **Container query**                              |
| Does the **page** chrome change with viewport?                             | **Media query** (`min-width`, mobile-first)      |
| Fluid type/spacing without breakpoint jumps?                               | **`clamp` / fluid units**                        |
| Hover affordances?                                                         | Gate with `(hover: hover)` and `(pointer: fine)` |
| Dark mode / reduced motion?                                                | `prefers-*` queries + tokens                     |

### Scenario: same card, different homes

A product card is full-width on a phone, half of main on desktop, and squeezed in a 280px sidebar widget. **Viewport** media queries can’t tell “sidebar vs main.” Give the parent a containment context:

```css
.widget,
.main {
  container-type: inline-size;
  container-name: card-slot;
}

@container card-slot (min-width: 400px) {
  .card {
    display: flex;
    gap: 16px;
  }
}
@container card-slot (max-width: 399px) {
  .card {
    display: block;
  }
}
```

Use `@media` for the site header collapsing, not for every card.

### Mobile-first page queries

```css
.card {
  padding: 16px;
}
@media (min-width: 768px) {
  .card {
    padding: 24px;
  }
}

@media (width <= 768px) {
} /* modern range syntax */
@media (hover: hover) and (pointer: fine) {
  .card:hover {
    transform: translateY(-2px);
  }
}
@media (prefers-color-scheme: dark) {
}
@media (prefers-reduced-motion: reduce) {
}
```

```css
h1 {
  font-size: clamp(1.5rem, 1rem + 2vw, 3rem);
}
.container {
  width: clamp(300px, 90vw, 1200px);
  margin-inline: auto;
}
```

### Self-check

1. Card breaks in a sidebar but looks fine on “mobile width” — MQ or CQ?
2. Write a fluid width: min 300, preferred 90vw, max 1200.

---

# Units

| Unit              | Resolves against                              | Use                                               |
| ----------------- | --------------------------------------------- | ------------------------------------------------- |
| `px`              | Absolute                                      | Borders, shadows, MQ thresholds                   |
| `rem`             | Root font-size                                | Type, spacing scales                              |
| `em`              | Element’s font-size (for `font-size`: parent) | Component-local padding that scales with its type |
| `%`               | Containing block (varies by property)         | Fluid widths                                      |
| `vw`/`vh`         | Viewport                                      | Full-bleed sections (prefer `dvh` for height)     |
| `dvh`/`svh`/`lvh` | Dynamic / small / large viewport              | Mobile browser chrome                             |
| `ch`              | Width of “0” glyph                            | Prose `max-width` (~65ch)                         |
| `fr`              | Grid free space                               | Grid tracks only                                  |

```css
.hero {
  height: 100vh; /* fallback */
  height: 100dvh; /* avoids mobile URL-bar overflow */
}
```

`em` compounds when nested for `font-size`; `rem` does not — that’s the usual review comment.

---

# Logical Properties

Physical (`top`/`left`/…) are screen-relative. Logical follow **writing mode**: **block** = stacking axis, **inline** = text axis. In horizontal LTR, block ≈ vertical, inline ≈ horizontal.

| Physical                   | Logical                                    |
| -------------------------- | ------------------------------------------ |
| `margin-top` / `bottom`    | `margin-block-start` / `end`               |
| `margin-left` / `right`    | `margin-inline-start` / `end`              |
| `width` / `height`         | `inline-size` / `block-size`               |
| `top` / `left`             | `inset-block-start` / `inset-inline-start` |
| `top/right/bottom/left: 0` | `inset: 0`                                 |
| `text-align: left`         | `text-align: start`                        |

```css
.container {
  max-inline-size: 1200px;
  margin-inline: auto;
}
.card {
  padding-block: 16px;
  padding-inline: 24px;
  border-inline-start: 4px solid blue; /* “start” edge in LTR and RTL */
}
```

In RTL, `margin-inline-start` is on the **right**. Prefer logical props in shared/UI-library CSS.

---

# Aspect Ratio

```css
.video {
  aspect-ratio: 16 / 9;
  width: 100%;
}
.thumb {
  aspect-ratio: 1;
  width: 100%;
  overflow: hidden;
}
.thumb img {
  width: 100%;
  height: 100%;
  object-fit: cover; /* hard crop; contain = letterbox */
}
```

Ratio is a **preference** — oversized intrinsic content can still grow the box unless you clip/`object-fit`. Pair with HTML `width`/`height` attributes to limit CLS. Common ratios: `1`, `16/9`, `4/3`, `21/9`.

---

# Colors and Custom Properties

CSS variables are **inherited runtime values**. They theme without a rebuild; Sass variables cannot.

### Token setup (decision pattern)

```css
:root {
  color-scheme: light dark; /* native controls/scrollbars adapt */
  accent-color: var(--color-primary);

  --hue: 250;
  --color-primary: oklch(60% 0.14 var(--hue));
  --color-primary-hover: color-mix(in oklch, var(--color-primary), black 15%);
  --bg: oklch(99% 0 0);
  --text: oklch(25% 0 0);
}

[data-theme='dark'] {
  --bg: oklch(20% 0.02 260);
  --text: oklch(95% 0 0);
}

@media (prefers-color-scheme: dark) {
  :root:not([data-theme='light']) {
    --bg: oklch(20% 0.02 260);
    --text: oklch(95% 0 0);
  }
}

.card {
  background: var(--bg);
  color: var(--text);
  border-color: var(--border, oklch(80% 0 0)); /* fallback */
}
```

OKLCH (or HSL channels) makes hover/ramps predictable; hex ramps are harder to derive by hand.

### Invalidation gotcha

If `--pad` computes to something invalid for `padding`, the whole `padding` declaration is dropped at computed-value time — **not** silently ignored as “bad var only.” Always ship fallbacks for critical tokens: `padding: var(--pad, 16px)`.

```css
@property --hue {
  syntax: '<number>';
  inherits: true;
  initial-value: 200;
} /* typed + animatable custom props */
```

```javascript
document.documentElement.style.setProperty('--hue', '20');
document.documentElement.dataset.theme = 'dark';
```

|                    | CSS variables | Sass variables |
| ------------------ | ------------- | -------------- |
| When               | Runtime       | Compile time   |
| Theme via JS/DOM   | Yes           | No             |
| Inherited/cascaded | Yes           | No             |

### Self-check

1. Why do CSS variables enable dark mode without rebuilding CSS?
2. What happens when `var(--x)` is invalid and has no fallback?

---

# Transitions and Animations

Animate **composite** properties (`transform`, `opacity`) to avoid layout thrash. Prefer specific transition lists over `all`. `will-change` only temporarily.

```css
.button {
  transition:
    background 0.3s ease,
    transform 0.2s ease;
}
.button:hover {
  transform: scale(1.05);
}

/* Not smoothly transitionable: display, font-family, position, content */
.modal {
  visibility: hidden;
  opacity: 0;
  transition:
    opacity 0.3s,
    visibility 0.3s;
}
.modal.active {
  visibility: visible;
  opacity: 1;
}
/* Entering from display:none → see Modern (@starting-style) */

@keyframes slideIn {
  from {
    transform: translateX(-100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}
.el {
  animation: slideIn 0.5s ease-out forwards;
}

@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

| Cheap                  | Expensive                                  |
| ---------------------- | ------------------------------------------ |
| `transform`, `opacity` | `width`, `height`, `top`, `left`, `margin` |

---

# Pseudo-classes and Pseudo-elements

**Pseudo-class** = state/structural match. **Pseudo-element** = fragment (`::before`, `::selection`).

### Essentials

```css
/* Keyboard-only focus ring */
button:focus {
  outline: none;
}
button:focus-visible {
  outline: 2px solid #4a90d9;
  outline-offset: 2px;
}

:is(h1, h2, h3) {
}
:where(.prose) a {
} /* 0 specificity base */
.card:has(img) {
}
:not(.active) {
}

li:nth-child(3n + 1) {
}
li:nth-child(-n + 5) {
} /* first 5 */
li:nth-of-type(odd) {
} /* among same tag — differs from :nth-child on mixed children */

input:required:invalid {
}
input:checked {
}
input:placeholder-shown {
}

.quote::before {
  content: '\201C';
} /* needs content */
::selection {
  background: #4a90d9;
  color: white;
}
li::marker {
  color: #4a90d9;
}
```

Prefer real DOM for complex chrome; `::before`/`::after` for light decoration. `::before` does nothing useful without `content`.

---

# Typography

```css
body {
  font-family: system-ui, 'Segoe UI', Roboto, sans-serif;
  line-height: 1.5; /* unitless — inherits as multiplier */
}
h1 {
  line-height: 1.2;
}

@font-face {
  font-family: 'MyFont';
  src: url('/fonts/myfont.woff2') format('woff2');
  font-weight: 400;
  font-display: swap; /* FOUT; use optional when CLS > branding */
}
```

| `font-display` | Behavior                             |
| -------------- | ------------------------------------ |
| `swap`         | Fallback immediately, swap in (FOUT) |
| `optional`     | Almost no late swap — best CLS       |
| `block`        | Invisible up to ~3s (FOIT)           |

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
.long-text {
  overflow-wrap: break-word;
  hyphens: auto;
}
.nums {
  font-variant-numeric: tabular-nums;
}
```

Unitless `line-height` inherits as a multiplier; `line-height: 24px` inherits `24px` absolutely — usually not what you want on nested sizes.

---

# Overflow and Scrolling

| Value     | Clips?             | Scroll container?                                              |
| --------- | ------------------ | -------------------------------------------------------------- |
| `visible` | No                 | No                                                             |
| `hidden`  | Yes                | Yes (JS can still scroll)                                      |
| `clip`    | Yes                | **No** — prefer when you only need clip (friendlier to sticky) |
| `auto`    | If needed          | Yes                                                            |
| `scroll`  | Always scrollports | Yes                                                            |

```css
html {
  scroll-behavior: smooth;
}
@media (prefers-reduced-motion: reduce) {
  html {
    scroll-behavior: auto;
  }
}

.carousel {
  display: flex;
  overflow-x: auto;
  scroll-snap-type: x mandatory;
}
.carousel-item {
  flex: 0 0 100%;
  scroll-snap-align: start;
}

.modal-body {
  overflow-y: auto;
  overscroll-behavior: contain; /* stop scroll chaining to the page */
}

.container {
  scrollbar-width: thin;
  scrollbar-color: #888 #f0f0f0;
}
```

`overflow: hidden` on an ancestor is a classic sticky breaker — see Positioning.

---

# CSS Functions

```css
width: calc(100% - 250px); /* spaces required around + and - */
width: min(90vw, 1200px);
width: max(300px, 50vw);
width: clamp(300px, 90vw, 1200px); /* min, preferred, max */
padding: env(safe-area-inset-top, 20px);

background: color-mix(in oklch, var(--primary), black 20%);
background: linear-gradient(135deg, #f00, #00f);
filter: blur(4px);
backdrop-filter: blur(10px);

.gradient-text {
  background: linear-gradient(135deg, #667eea, #764ba2);
  background-clip: text;
  color: transparent;
}
```

Fluid type/width with `clamp` also lives under Responsive. Safe-area `env()` matters for notched phones / home indicators.

---

# Modern CSS Features

Index of newer tools — deepen individually when a role demands them.

```css
@layer reset, base, components, utilities;

.card {
  padding: 16px;
  .title {
    font-size: 1.5rem;
  }
  &:hover {
    box-shadow: 0 2px 8px rgb(0 0 0 / 10%);
  }
  @media (width <= 768px) {
    padding: 8px;
  }
}

@scope (.card) to (.card-footer) {
  h2 {
    font-size: 1.5rem;
  }
}

/* Animate in from display: none */
.modal {
  display: none;
  opacity: 0;
  transition:
    opacity 0.3s,
    display 0.3s allow-discrete;
  @starting-style {
    opacity: 0;
  }
}
.modal.open {
  display: block;
  opacity: 1;
}

@supports selector(:has(*)) {
  .card:has(img) {
    padding: 0;
  }
}

::view-transition-old(root) {
  animation: fade-out 0.3s;
}
::view-transition-new(root) {
  animation: fade-in 0.3s;
}
```

Unlayered styles beat layered styles. `@scope` limits leak without Modules; Modules still win for build-time isolation in component apps.

---

# Performance

Cost ladder for style changes:

```
Reflow (layout) → Repaint → Composite
Most expensive ←————————————→ Least

Reflow:  width, height, margin, padding, font-size, reading offsetWidth
Repaint: color, background, box-shadow, visibility
Composite: transform, opacity
```

### Layout thrashing

```javascript
// BAD — read/write alternation forces layout each iteration
for (const el of elements) {
  el.style.width = container.offsetWidth + 'px';
}

// GOOD — batch reads, then writes
const width = container.offsetWidth;
for (const el of elements) {
  el.style.width = width + 'px';
}

// ALSO GOOD — double rAF or ResizeObserver when measuring after paint
```

### Containment — when and when not

```css
.card {
  contain: content; /* layout + paint isolation — good for repeated cards */
}
.section {
  content-visibility: auto;
  contain-intrinsic-size: 0 500px; /* placeholder size avoids scrollbar jump */
}
```

**When not to:** don’t `contain: strict` on a box whose children must influence outside layout (e.g. expanding accordion measured by a parent). `content-visibility: auto` shines on long feeds; it’s wasted on above-the-fold hero content.

### Checklist

```
Animate transform/opacity
Batch DOM reads, then writes
contain: content on card-like islands
content-visibility: auto on long below-the-fold lists
will-change only moments before animation — never globally
Inline critical CSS for first paint
```

### Self-check

1. Rank animating `opacity`, `left`, `background-color` from cheapest to costliest.
2. What is layout thrashing?
3. When is `content-visibility: auto` the wrong tool?

---

# Architecture

Architecture is **ownership + collision control + change velocity** — not “which tool won Twitter.”

### Ownership matrix

| Approach        | Collision control                | Velocity            | Failure mode                                |
| --------------- | -------------------------------- | ------------------- | ------------------------------------------- |
| **BEM**         | Naming convention in global CSS  | Medium              | Verbose; discipline-dependent               |
| **CSS Modules** | Compile-time local scope         | Medium-high         | Sharing tokens/cross-cuts needs convention  |
| **Tailwind**    | Utilities + lint                 | Very high           | Design soup without constraints; HTML noise |
| **CSS-in-JS**   | Colocated with component         | High for dynamic    | Runtime/tooling cost; duplication risk      |
| **`@layer`**    | Cascade priority between systems | Integrator          | Mis-ordered layers recreate !important wars |
| **Shadow DOM**  | True encapsulation               | Low-medium for apps | Theming/piercing; heavier for whole apps    |

### Practical defaults (interview framing)

- **Component React/Vue app:** CSS Modules or Tailwind + shared CSS variable tokens.
- **Multi-team legacy global CSS:** BEM (or similar) + `@layer` at vendor boundaries.
- **Design system / widgets:** Shadow DOM or Modules + tokens; document escape hatches.
- **Never:** three token sources (Figma hex, Sass `$`, random CSS vars) without a single pipeline.

```css
@layer reset, tokens, vendor, components, utilities, overrides;
/* third-party → vendor; app → components; utilities last among layered */
```

```css
.card {
}
.card__title {
}
.card--featured {
} /* BEM: keep selectors ~0-1-0 */
```

**Token rule:** components consume `var(--color-primary)`, not raw hex. Themes and JS become one `setProperty` / `data-theme` away.

In interviews: state the **problem** (ownership, collisions, speed), pick a default, name the **escape hatch** (`@layer`, unlayered override, `revert-layer`). Don’t recite a winner.

### Self-check

1. What precise problem does BEM solve?
2. When do Modules beat BEM in the same React codebase?
3. Where does third-party CSS sit in an `@layer` stack and why?

---

## Quick revisit map

| Situation               | Reach for                                              |
| ----------------------- | ------------------------------------------------------ |
| Layout bugs             | Formatting context + containing block                  |
| Won’t shrink / truncate | Intrinsic min size / `min-width: 0` / `minmax(0, 1fr)` |
| Override fights         | Layer → specificity → order + keywords                 |
| Flex weirdness          | Axes + basis → shrink → grow                           |
| Grid weirdness          | Lines/tracks + explicit vs implicit + fill/fit         |
| Janky UI                | Reflow vs composite                                    |
| Structure debate        | Ownership + collision + tokens                         |

|
