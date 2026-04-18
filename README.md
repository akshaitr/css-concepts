# CSS Concepts

1. [Box Model](#box-model)
2. [Selectors and Specificity](#selectors-and-specificity)
3. [Positioning](#positioning)
4. [Display and Layout](#display-and-layout)
5. [Flexbox](#flexbox)
6. [CSS Grid](#css-grid)
7. [Responsive Design](#responsive-design)
8. [Units](#units)
9. [Colors and Custom Properties](#colors-and-custom-properties)
10. [Transitions and Animations](#transitions-and-animations)
11. [Pseudo-classes and Pseudo-elements](#pseudo-classes-and-pseudo-elements)
12. [Typography](#typography)
13. [Overflow and Scrolling](#overflow-and-scrolling)
14. [CSS Functions](#css-functions)
15. [Modern CSS Features](#modern-css-features)
16. [Performance](#performance)
17. [BEM and Architecture](#bem-and-architecture)

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
  margin: 20px;             /* all sides */
  margin: 10px 20px;        /* vertical | horizontal */
  margin: 10px 20px 30px;   /* top | horizontal | bottom */
  margin: 10px 20px 30px 40px; /* top | right | bottom | left (clockwise) */
  
  margin: auto;             /* center horizontally (on block elements with a width) */
  margin-inline: auto;      /* center using logical properties */
}
```

### Margin collapsing

When two vertical margins touch, they don't add up — the larger one wins. This only happens vertically, never horizontally.

```css
.box1 {
  margin-bottom: 30px;
}

.box2 {
  margin-top: 20px;
}

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
    <!-- Parent appears to have 40px top margin -->
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
  outline-offset: 4px; /* space between outline and border */
  /* Element size doesn't change */
}
```

Outline is commonly used for focus indicators. Never remove outline without providing an alternative:

```css
/* BAD — removes focus indicator, breaks keyboard accessibility */
*:focus { outline: none; }

/* GOOD — custom focus indicator */
*:focus-visible {
  outline: 2px solid #4A90D9;
  outline-offset: 2px;
}
```

# Selectors and Specificity

### Selector types

```css
/* Universal */
* { margin: 0; }

/* Type (element) */
p { color: black; }
h1 { font-size: 2rem; }

/* Class */
.card { padding: 16px; }
.card.featured { border: 2px solid gold; } /* both classes on same element */

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

Specificity determines which CSS rule wins when multiple rules target the same element. It's calculated as a three-part score:

```
(ID) - (Class/Attribute/Pseudo-class) - (Element/Pseudo-element)

Examples:
p                    → 0-0-1
.card                → 0-1-0
#header              → 1-0-0
p.card               → 0-1-1
#header .nav a       → 1-1-1
#header .nav a:hover → 1-2-1

/* Compare left to right: */
#header    (1-0-0) beats .card.featured (0-2-0)
/* because 1 ID > any number of classes */
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

### The cascade — how CSS decides which rule applies

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
   - Higher specificity wins

3. Source order (if same specificity)
   - Last rule in the stylesheet wins
```

### !important — and why to avoid it

```css
.text {
  color: red !important; /* wins over everything except another !important with higher specificity */
}

#header .text {
  color: blue; /* loses to !important above, even with higher specificity */
}

#header .text {
  color: green !important; /* wins because same importance + higher specificity */
}
```

`!important` breaks the natural cascade and makes debugging difficult. Legitimate uses are rare — overriding third-party CSS you can't modify, or utility classes in frameworks.

### Modern pseudo-class selectors

**`:is()` — matches any selector in the list, takes highest specificity of its arguments:**

```css
/* Without :is() */
.card h1,
.card h2,
.card h3 {
  color: navy;
}

/* With :is() — same result, less code */
.card :is(h1, h2, h3) {
  color: navy;
}

/* Specificity = highest selector inside :is() */
:is(#header, .nav) a { }
/* Specificity: 1-0-1 (takes #header's specificity) */
```

**`:where()` — same as :is() but with ZERO specificity:**

```css
/* Zero specificity — easy to override */
:where(.card, .panel) h2 {
  color: navy;
}

/* This easily overrides the above because :where() has 0 specificity */
h2 {
  color: red;
}
```

Use `:where()` for base/default styles that should be easily overridable.

**`:has()` — parent selector (the most powerful modern selector):**

```css
/* Style a card that contains an image */
.card:has(img) {
  padding: 0;
}

/* Style a label when its input is focused */
label:has(+ input:focus) {
  color: blue;
}

/* Style the page based on what it contains */
body:has(.modal.open) {
  overflow: hidden;
}

/* Style a form when it has invalid inputs */
form:has(:invalid) {
  border-color: red;
}
```

`:has()` was the most requested CSS feature for decades. It lets you select elements based on their children or siblings — previously impossible without JavaScript.

# Positioning

### static (default)

```css
.box {
  position: static;
  /* top, right, bottom, left, z-index have NO effect */
  /* Element follows normal document flow */
}
```

### relative

```css
.box {
  position: relative;
  top: 20px;
  left: 10px;
  /* Offset from its NORMAL position */
  /* Original space is preserved — other elements don't shift */
  /* Creates a new stacking context for z-index */
  /* Becomes the containing block for absolute children */
}
```

### absolute

```css
.box {
  position: absolute;
  top: 0;
  right: 0;
  /* Removed from normal flow — other elements ignore it */
  /* Positioned relative to nearest positioned ancestor */
  /* If no positioned ancestor → relative to the viewport (initial containing block) */
}
```

The classic pattern — absolute inside relative:

```css
.parent {
  position: relative;  /* becomes the reference point */
}

.badge {
  position: absolute;
  top: -8px;
  right: -8px;        /* positioned relative to .parent */
}
```

### fixed

```css
.navbar {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  /* Removed from normal flow */
  /* Positioned relative to the VIEWPORT */
  /* Stays in place during scrolling */
  /* ⚠️ Breaks if an ancestor has transform, filter, or perspective */
}
```

### sticky

```css
.section-header {
  position: sticky;
  top: 0;
  /* Behaves like relative until scroll reaches the threshold */
  /* Then behaves like fixed within its parent container */
  /* Stops being fixed when parent scrolls out of view */
}
```

```css
/* Common gotcha — sticky doesn't work if: */
.parent {
  overflow: hidden; /* or auto/scroll — sticky child can't escape */
}
```

**Sticky requires:**
- A threshold value (top, bottom, left, or right)
- The parent must have scrollable content (be taller than viewport)
- No `overflow: hidden/auto/scroll` on any ancestor between the sticky element and the scroll container

### Stacking context and z-index

`z-index` only works on positioned elements (relative, absolute, fixed, sticky) and flex/grid children.

```css
.box1 { position: relative; z-index: 10; }
.box2 { position: relative; z-index: 5; }
/* box1 appears on top of box2 */
```

**What creates a new stacking context:**
- `position: relative/absolute/fixed/sticky` with `z-index` other than `auto`
- `opacity` less than 1
- `transform`, `filter`, `perspective`, `clip-path`
- `isolation: isolate`
- `will-change` with certain values
- Flex/grid children with `z-index` other than `auto`

**The trap:** z-index only competes within the same stacking context. A child with `z-index: 9999` can never appear above an element in a higher parent stacking context.

```css
.parent-a { position: relative; z-index: 1; }
.parent-b { position: relative; z-index: 2; }

.child-of-a { position: relative; z-index: 9999; }
/* Still appears BEHIND .parent-b because parent-a's 
   stacking context (z-index: 1) is lower than parent-b's (2) */
```

**Fix z-index wars with `isolation`:**

```css
.component {
  isolation: isolate; /* creates a new stacking context without side effects */
}
```

### Centering techniques

```css
/* Flexbox — most common */
.parent {
  display: flex;
  justify-content: center;
  align-items: center;
}

/* Grid — shortest */
.parent {
  display: grid;
  place-items: center;
}

/* Absolute + transform — when flex/grid isn't an option */
.child {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

/* Margin auto — horizontal centering of block elements */
.child {
  width: 500px;
  margin-inline: auto;
}

/* Grid + margin auto — both axes */
.parent {
  display: grid;
}
.child {
  margin: auto;
}
```

# Display and Layout

### Block elements

```css
/* Block elements: */
/* - Take full available width */
/* - Start on a new line */
/* - Respect width, height, margin, padding on all sides */
/* Examples: div, p, h1-h6, section, article, header, footer, form, ul, li */

div {
  display: block;
  width: 50%;     /* works */
  height: 100px;  /* works */
  margin: 20px;   /* works on all sides */
}
```

### Inline elements

```css
/* Inline elements: */
/* - Take only as much width as their content */
/* - Don't start on a new line */
/* - Ignore width and height */
/* - Vertical margin and padding don't push other elements */
/* Examples: span, a, strong, em, img (replaced), code, label */

span {
  display: inline;
  width: 200px;         /* IGNORED */
  height: 100px;        /* IGNORED */
  margin-top: 20px;     /* IGNORED */
  margin-left: 20px;    /* works (horizontal) */
  padding-top: 20px;    /* applies visually but doesn't push elements below */
}
```

### Inline-block

```css
/* Best of both worlds: */
/* - Sits inline with text (doesn't start new line) */
/* - Respects width, height, margin, padding on all sides */

.badge {
  display: inline-block;
  width: 100px;          /* works */
  height: 30px;          /* works */
  margin: 10px;          /* works on all sides */
  vertical-align: middle;
}
```

### display: none vs visibility: hidden

```css
/* display: none — completely removed from layout */
.hidden {
  display: none;
  /* No space taken, not accessible, not interactive */
}

/* visibility: hidden — invisible but space preserved */
.invisible {
  visibility: hidden;
  /* Space still taken, not accessible, not interactive */
  /* Child elements CAN override with visibility: visible */
}
```

### display: contents

```css
/* Removes the element's box but keeps its children in the layout */
.wrapper {
  display: contents;
  /* The wrapper element "disappears" but its children 
     participate in the parent's layout as if the wrapper wasn't there */
}
```

Useful when you need a semantic wrapper but don't want it to affect flex/grid layout:

```html
<div class="grid">
  <div class="wrapper" style="display: contents;">
    <div>Item 1</div>  <!-- participates in .grid's grid layout directly -->
    <div>Item 2</div>
  </div>
</div>
```

### Flow root

```css
.container {
  display: flow-root;
  /* Creates a new block formatting context (BFC) */
  /* Contains floats */
  /* Prevents margin collapsing with children */
  /* Modern replacement for the clearfix hack */
}
```

# Flexbox

Flexbox is a one-dimensional layout system — it handles either a row OR a column at a time.

### Container properties

```css
.container {
  display: flex;           /* or inline-flex */
  
  /* Direction */
  flex-direction: row;            /* default: left to right */
  flex-direction: row-reverse;    /* right to left */
  flex-direction: column;         /* top to bottom */
  flex-direction: column-reverse; /* bottom to top */
  
  /* Wrapping */
  flex-wrap: nowrap;  /* default: all items on one line, may shrink */
  flex-wrap: wrap;    /* items wrap to new lines */
  
  /* Shorthand */
  flex-flow: row wrap;
  
  /* Main axis alignment (direction items are flowing) */
  justify-content: flex-start;     /* default */
  justify-content: flex-end;
  justify-content: center;
  justify-content: space-between;  /* equal space between items */
  justify-content: space-around;   /* equal space around items */
  justify-content: space-evenly;   /* truly equal spacing */
  
  /* Cross axis alignment (perpendicular to main axis) */
  align-items: stretch;     /* default: items stretch to fill container */
  align-items: flex-start;
  align-items: flex-end;
  align-items: center;
  align-items: baseline;    /* aligns text baselines */
  
  /* Multi-line cross axis alignment (only with flex-wrap: wrap) */
  align-content: flex-start;
  align-content: center;
  align-content: space-between;
  
  /* Gap between items */
  gap: 16px;           /* row and column gap */
  row-gap: 16px;
  column-gap: 8px;
}
```

### Item properties

```css
.item {
  /* Grow — how much extra space this item should take */
  flex-grow: 0;    /* default: don't grow */
  flex-grow: 1;    /* take equal share of extra space */
  
  /* Shrink — how much this item should shrink when space is tight */
  flex-shrink: 1;  /* default: shrink equally */
  flex-shrink: 0;  /* don't shrink — maintain size */
  
  /* Basis — starting size before grow/shrink */
  flex-basis: auto;   /* default: use width/height */
  flex-basis: 200px;  /* start at 200px */
  flex-basis: 0;      /* ignore content size, distribute purely by grow */
  
  /* Shorthand (grow | shrink | basis) */
  flex: 0 1 auto;  /* default */
  flex: 1;         /* same as: flex: 1 1 0 — grow equally, ignore content size */
  flex: auto;      /* same as: flex: 1 1 auto — grow equally, respect content size */
  flex: none;      /* same as: flex: 0 0 auto — rigid, no grow/shrink */
  
  /* Override cross-axis alignment for individual item */
  align-self: center;
  align-self: flex-end;
  
  /* Order — change visual order without changing HTML */
  order: 0;   /* default */
  order: -1;  /* move to start */
  order: 1;   /* move to end */
}
```

### flex: 1 vs flex: auto

This distinction matters in practice:

```css
/* flex: 1 (flex: 1 1 0) */
/* basis is 0 — ignores content size */
/* All items get exactly equal width regardless of content */
.item { flex: 1; }

/* flex: auto (flex: 1 1 auto) */
/* basis is auto — starts from content size */
/* Items with more content get more space */
.item { flex: auto; }
```

### Common flexbox patterns

```css
/* Navbar — logo left, links right */
.navbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

/* Center a single item */
.container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
}

/* Footer pushed to bottom */
body {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}
main { flex: 1; }  /* main grows, footer stays at bottom */

/* Equal height cards */
.card-grid {
  display: flex;
  gap: 16px;
}
.card {
  flex: 1;           /* equal width */
  display: flex;
  flex-direction: column;
}
.card-body { flex: 1; }  /* body grows, buttons align at bottom */
.card-footer { margin-top: auto; }

/* Input with button */
.input-group {
  display: flex;
}
.input-group input { flex: 1; }   /* input takes remaining space */
.input-group button { flex: none; } /* button keeps its size */
```

### Flexbox gotchas

```css
/* 1. min-width: auto — flex items won't shrink below content size by default */
.item {
  flex: 1;
  min-width: 0;        /* fix: allow shrinking below content */
  overflow: hidden;     /* or: clip the overflow */
}

/* This is why text doesn't truncate in flex items without min-width: 0 */
.item {
  flex: 1;
  min-width: 0;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;  /* now truncation works */
}

/* 2. Images stretch in flex containers */
img {
  align-self: flex-start; /* fix: don't stretch */
  /* or */ max-width: 100%;
}

/* 3. margin: auto in flexbox absorbs extra space */
.item:last-child {
  margin-left: auto; /* pushes this item to the far right */
}
```

# CSS Grid

Grid is a two-dimensional layout system — it handles rows AND columns simultaneously.

### Container properties

```css
.grid {
  display: grid;  /* or inline-grid */
  
  /* Define columns */
  grid-template-columns: 200px 1fr 200px;        /* 3 columns: fixed-fluid-fixed */
  grid-template-columns: repeat(3, 1fr);          /* 3 equal columns */
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr)); /* responsive */
  
  /* Define rows */
  grid-template-rows: 60px 1fr 40px;    /* header-main-footer */
  grid-template-rows: auto;              /* rows size to content */
  
  /* Gap */
  gap: 16px;
  row-gap: 16px;
  column-gap: 8px;
  
  /* Alignment — for ALL items */
  justify-items: stretch;   /* default: items fill their cell horizontally */
  justify-items: start;
  justify-items: center;
  justify-items: end;
  
  align-items: stretch;     /* default: items fill their cell vertically */
  align-items: start;
  align-items: center;
  align-items: end;
  
  /* Alignment — for the GRID inside the container */
  justify-content: center;        /* centers the grid horizontally */
  align-content: center;          /* centers the grid vertically */
  place-content: center;          /* both */
  
  /* place-items shorthand */
  place-items: center;            /* align-items + justify-items */
}
```

### Grid template areas

```css
.layout {
  display: grid;
  grid-template-columns: 200px 1fr 200px;
  grid-template-rows: 60px 1fr 40px;
  grid-template-areas:
    "header  header  header"
    "sidebar content aside"
    "footer  footer  footer";
  gap: 16px;
  min-height: 100vh;
}

.header  { grid-area: header; }
.sidebar { grid-area: sidebar; }
.content { grid-area: content; }
.aside   { grid-area: aside; }
.footer  { grid-area: footer; }

/* Responsive — change layout with one property */
@media (max-width: 768px) {
  .layout {
    grid-template-columns: 1fr;
    grid-template-areas:
      "header"
      "content"
      "sidebar"
      "aside"
      "footer";
  }
}
```

### Item placement

```css
.item {
  /* Line-based placement */
  grid-column: 1 / 3;       /* spans from line 1 to line 3 (2 columns) */
  grid-row: 1 / 2;          /* spans from line 1 to line 2 (1 row) */
  
  /* Span syntax */
  grid-column: span 2;      /* spans 2 columns from wherever placed */
  grid-row: span 3;         /* spans 3 rows */
  
  /* Shorthand */
  grid-area: 1 / 1 / 3 / 3; /* row-start / col-start / row-end / col-end */
  
  /* Named area */
  grid-area: header;
  
  /* Self alignment — override container alignment for this item */
  justify-self: center;
  align-self: end;
  place-self: center end;
}
```

### auto-fill vs auto-fit

```css
/* auto-fill — creates as many tracks as fit, even if empty */
grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
/* If container is 1000px: creates 5 columns (some may be empty) */

/* auto-fit — collapses empty tracks, items stretch to fill */
grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
/* If container is 1000px with 2 items: items stretch across full width */
```

Use `auto-fit` for most responsive grids — items expand to fill available space.
Use `auto-fill` when you want to maintain consistent column sizes even with few items.

### Explicit vs implicit grid

```css
.grid {
  display: grid;
  /* Explicit grid — you defined these */
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: 100px 100px;
  
  /* Implicit grid — browser creates these for overflow items */
  grid-auto-rows: 150px;         /* height of auto-created rows */
  grid-auto-columns: 100px;      /* width of auto-created columns */
  grid-auto-flow: row;           /* default: fill row by row */
  grid-auto-flow: column;        /* fill column by column */
  grid-auto-flow: dense;         /* fill gaps by reordering items */
}
```

### Subgrid

Lets child grids align to the parent grid's tracks:

```css
.parent {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 16px;
}

.child {
  grid-column: span 2;
  display: grid;
  grid-template-columns: subgrid; /* inherits parent's column tracks */
  /* child's items align perfectly with parent's grid lines */
}
```

### Common grid patterns

```css
/* Responsive card grid — no media queries needed */
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 24px;
}

/* Holy grail layout */
.page {
  display: grid;
  grid-template: 
    "header header header" 60px
    "nav    main   aside"  1fr
    "footer footer footer" 40px
    / 200px 1fr    200px;
  min-height: 100vh;
}

/* Masonry-like (equal columns, auto rows) */
.masonry {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-auto-rows: auto;
  gap: 16px;
}

/* Full bleed within constrained layout */
.content {
  display: grid;
  grid-template-columns: 
    1fr 
    min(65ch, 100%) 
    1fr;
}
.content > * {
  grid-column: 2;
}
.full-bleed {
  grid-column: 1 / -1; /* spans all columns */
}
```

### Flexbox vs Grid — when to use which

```
Use Flexbox when:                    Use Grid when:
- One dimension (row OR column)      - Two dimensions (rows AND columns)
- Content dictates layout            - Layout dictates content
- Alignment in a single axis         - Items need to align in both axes
- Navigation bars                    - Page layouts
- Centering content                  - Card grids
- Input groups                       - Dashboard layouts
- Equal height columns               - Complex overlapping designs
```

In practice, you use both together — grid for page layout, flex for component internals.

# Responsive Design

### Media queries

```css
/* Width-based (most common) */
@media (max-width: 768px) { }   /* up to 768px */
@media (min-width: 769px) { }   /* 769px and above */

/* Range syntax (modern) */
@media (width <= 768px) { }
@media (768px <= width <= 1024px) { }

/* Combine conditions */
@media (min-width: 768px) and (max-width: 1024px) { }

/* Device features */
@media (hover: hover) { }           /* device has hover capability */
@media (hover: none) { }            /* touch devices */
@media (prefers-color-scheme: dark) { } /* dark mode */
@media (prefers-reduced-motion: reduce) { } /* user prefers less animation */
@media (orientation: landscape) { }
@media (pointer: coarse) { }        /* imprecise pointer (touch) */
@media (pointer: fine) { }          /* precise pointer (mouse) */
```

### Mobile-first vs Desktop-first

```css
/* Mobile-first (recommended) — base styles for mobile, enhance upward */
.card { padding: 16px; }

@media (min-width: 768px) {
  .card { padding: 24px; display: flex; }
}

@media (min-width: 1200px) {
  .card { padding: 32px; max-width: 1200px; }
}

/* Desktop-first — base styles for desktop, reduce downward */
.card { padding: 32px; max-width: 1200px; display: flex; }

@media (max-width: 1199px) {
  .card { max-width: none; }
}

@media (max-width: 767px) {
  .card { padding: 16px; display: block; }
}
```

Mobile-first is preferred because: mobile styles are simpler (less CSS to start), progressive enhancement is more robust than graceful degradation, and it forces you to prioritize content.

### Container queries

Media queries respond to the **viewport**. Container queries respond to the **parent container's size**. This is a game-changer for reusable components.

```css
/* Define a containment context */
.card-container {
  container-type: inline-size;
  container-name: card;  /* optional: name the container */
}

/* Query the container's width */
@container card (min-width: 400px) {
  .card {
    display: flex;
    gap: 16px;
  }
}

@container card (min-width: 600px) {
  .card {
    grid-template-columns: 1fr 2fr;
  }
}

/* Without a name — queries the nearest ancestor container */
@container (min-width: 300px) {
  .card-title { font-size: 1.5rem; }
}
```

### Fluid typography with clamp()

```css
/* Font size scales smoothly between 1rem and 2rem based on viewport */
h1 {
  font-size: clamp(1.5rem, 4vw, 3rem);
  /* minimum: 1.5rem (24px) */
  /* preferred: 4vw (scales with viewport) */
  /* maximum: 3rem (48px) */
}

/* Fluid spacing */
.section {
  padding: clamp(1rem, 5vw, 4rem);
}

/* Fluid container width */
.container {
  width: clamp(300px, 90vw, 1200px);
  margin-inline: auto;
}
```

### Common breakpoints

```css
/* There's no universal standard, but these are common: */
/* Mobile:  < 768px   */
/* Tablet:  768-1024px */
/* Desktop: > 1024px   */
/* Large:   > 1440px   */

/* Better approach: set breakpoints where YOUR content breaks, 
   not based on device widths */
```

# Units

### Absolute units

```css
.box {
  width: 200px;   /* pixels — most common, 1px = 1 device pixel (at 1x) */
  /* cm, mm, in, pt — print-only, avoid for screens */
}
```

### Relative units

```css
.text {
  /* em — relative to the element's own font-size (or parent's for font-size) */
  font-size: 1.5em;    /* 1.5x parent's font-size */
  padding: 1em;        /* 1x THIS element's font-size */
  
  /* rem — relative to root (<html>) font-size. Default root = 16px */
  font-size: 1.5rem;   /* 1.5 × 16px = 24px */
  padding: 1rem;       /* 16px, always predictable */
  
  /* % — relative to parent */
  width: 50%;          /* 50% of parent's width */
  font-size: 120%;     /* 120% of parent's font-size */
  
  /* ch — width of the "0" character in the current font */
  max-width: 65ch;     /* roughly 65 characters per line — ideal for readability */
}
```

### Viewport units

```css
.hero {
  /* Basic viewport units */
  height: 100vh;       /* 100% of viewport height */
  width: 100vw;        /* 100% of viewport width */
  font-size: 5vmin;    /* 5% of the smaller dimension */
  font-size: 5vmax;    /* 5% of the larger dimension */
  
  /* New viewport units (solve the mobile address bar problem) */
  height: 100dvh;      /* dynamic: changes when address bar shows/hides */
  height: 100svh;      /* small: viewport with address bar visible */
  height: 100lvh;      /* large: viewport with address bar hidden */
}
```

**The mobile viewport problem:** On mobile browsers, `100vh` is the large viewport (address bar hidden). When the address bar is visible, content overflows. `100dvh` fixes this by updating dynamically.

### When to use which unit

```
Unit  | Use for
------|------------------------------------------
px    | Borders, shadows, small fixed values, media queries
rem   | Font sizes, spacing, widths — anything that should scale with root
em    | Component-internal spacing that should scale with the component's font
%     | Fluid widths relative to parent
vw/vh | Full-page sections, viewport-relative sizing
dvh   | Full-height sections on mobile
ch    | Max-width for readable text (65ch)
fr    | Grid track sizing
```

**General rule:** Use `rem` for most things. Use `px` for borders and tiny values. Use `%` or `fr` for fluid layouts.

# Colors and Custom Properties

### Color formats

```css
.box {
  /* Named colors */
  color: red;
  color: rebeccapurple;
  
  /* Hex */
  color: #ff0000;         /* full hex */
  color: #f00;            /* shorthand */
  color: #ff000080;       /* with alpha (50%) */
  
  /* RGB */
  color: rgb(255, 0, 0);
  color: rgb(255 0 0);              /* modern: space-separated */
  color: rgb(255 0 0 / 50%);        /* with alpha */
  
  /* HSL — often easier to work with */
  color: hsl(0, 100%, 50%);         /* red: hue 0°, full saturation, 50% lightness */
  color: hsl(0 100% 50%);           /* modern: space-separated */
  color: hsl(0 100% 50% / 50%);     /* with alpha */
  
  /* Modern color spaces */
  color: oklch(70% 0.15 30);        /* perceptually uniform lightness */
  color: color(display-p3 1 0 0);   /* wider gamut */
}
```

**HSL is intuitive:**
- **Hue:** 0° = red, 120° = green, 240° = blue (color wheel)
- **Saturation:** 0% = gray, 100% = vivid
- **Lightness:** 0% = black, 50% = pure color, 100% = white

Creating color variations with HSL is trivial:
```css
:root {
  --primary-h: 220;
  --primary-s: 90%;
  --primary: hsl(var(--primary-h) var(--primary-s) 50%);
  --primary-light: hsl(var(--primary-h) var(--primary-s) 70%);
  --primary-dark: hsl(var(--primary-h) var(--primary-s) 30%);
}
```

### CSS Custom Properties (Variables)

```css
/* Define on :root for global scope */
:root {
  --color-primary: #4A90D9;
  --color-text: #333;
  --spacing-sm: 8px;
  --spacing-md: 16px;
  --spacing-lg: 24px;
  --font-body: 'Inter', system-ui, sans-serif;
  --radius: 8px;
  --shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

/* Use them */
.card {
  color: var(--color-text);
  padding: var(--spacing-md);
  border-radius: var(--radius);
  box-shadow: var(--shadow);
  font-family: var(--font-body);
}

/* Fallback value */
.box {
  color: var(--undefined-variable, #333); /* uses #333 if variable doesn't exist */
}
```

### Scoping and overriding

```css
:root {
  --bg: white;
  --text: black;
}

/* Override for dark mode */
@media (prefers-color-scheme: dark) {
  :root {
    --bg: #1a1a2e;
    --text: #e0e0e0;
  }
}

/* Override for a specific component */
.dark-section {
  --bg: #1a1a2e;
  --text: #e0e0e0;
}

/* Component uses the variables — automatically adapts */
.card {
  background: var(--bg);
  color: var(--text);
}
```

Custom properties cascade and inherit just like any other CSS property. A variable set on a parent is available to all descendants.

### Custom properties vs preprocessor variables (Sass)

```
Feature                  | CSS Variables        | Sass Variables
-------------------------|---------------------|------------------
Evaluated at             | Runtime (browser)   | Compile time (build)
Can be changed by JS     | ✅ Yes              | ❌ No
Can be scoped/inherited  | ✅ Yes              | ❌ No (global or local)
Work in media queries    | ✅ Yes              | ✅ Yes
Need a build step        | ❌ No               | ✅ Yes
IE11 support             | ❌ No               | ✅ Yes (compiled to values)
```

```javascript
// Changing CSS variables from JavaScript
document.documentElement.style.setProperty('--color-primary', '#ff0000');

// Reading
getComputedStyle(document.documentElement).getPropertyValue('--color-primary');
```

# Transitions and Animations

### Transitions

Transitions animate the change between two states (e.g., hover, focus, class toggle).

```css
.button {
  background: #4A90D9;
  transform: scale(1);
  
  /* Shorthand: property | duration | timing | delay */
  transition: all 0.3s ease;
  
  /* Individual properties (preferred — more performant than 'all') */
  transition: background 0.3s ease, transform 0.2s ease;
  
  /* Multiple transitions */
  transition-property: background, transform;
  transition-duration: 0.3s, 0.2s;
  transition-timing-function: ease, ease-out;
  transition-delay: 0s, 0.1s;
}

.button:hover {
  background: #357ABD;
  transform: scale(1.05);
}
```

### Timing functions

```css
.box {
  transition-timing-function: linear;       /* constant speed */
  transition-timing-function: ease;         /* default: slow-fast-slow */
  transition-timing-function: ease-in;      /* starts slow */
  transition-timing-function: ease-out;     /* ends slow */
  transition-timing-function: ease-in-out;  /* slow start and end */
  
  /* Custom cubic-bezier */
  transition-timing-function: cubic-bezier(0.68, -0.55, 0.265, 1.55); /* bounce effect */
  
  /* Steps — for sprite animations */
  transition-timing-function: steps(4, jump-start);
}
```

### What CAN'T be transitioned

```css
/* These properties CANNOT be animated/transitioned: */
/* display, font-family, position, float, grid-template-columns (values), content */

/* Common mistake: */
.modal {
  display: none;
  opacity: 0;
  transition: opacity 0.3s;
}
.modal.active {
  display: block;    /* transition won't work — display snaps instantly */
  opacity: 1;
}

/* Fix: use visibility instead of display, or use @starting-style */
.modal {
  visibility: hidden;
  opacity: 0;
  transition: opacity 0.3s, visibility 0.3s;
}
.modal.active {
  visibility: visible;
  opacity: 1;
}
```

### Keyframe animations

For complex, multi-step, or looping animations:

```css
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

/* Multi-step animation */
@keyframes pulse {
  0%   { transform: scale(1); }
  50%  { transform: scale(1.1); }
  100% { transform: scale(1); }
}

.element {
  /* Shorthand: name | duration | timing | delay | iteration | direction | fill | play-state */
  animation: slideIn 0.5s ease-out;
  
  /* Individual properties */
  animation-name: slideIn;
  animation-duration: 0.5s;
  animation-timing-function: ease-out;
  animation-delay: 0.2s;
  animation-iteration-count: 1;        /* or infinite */
  animation-direction: normal;          /* or reverse, alternate, alternate-reverse */
  animation-fill-mode: forwards;        /* keeps end state after animation */
  animation-play-state: running;        /* or paused */
}
```

### animation-fill-mode

```css
/* none (default) — element snaps back to original state after animation */
/* forwards — element keeps the final keyframe state */
/* backwards — element applies first keyframe state during delay */
/* both — applies both forwards and backwards */

.fade-in {
  opacity: 0;                        /* initial state */
  animation: fadeIn 0.5s ease 1s;    /* 1s delay */
  animation-fill-mode: both;
  /* backwards: opacity 0 during 1s delay */
  /* forwards: opacity 1 after animation */
}

@keyframes fadeIn {
  to { opacity: 1; }
}
```

### Transform

Transforms change an element's visual appearance without affecting layout (no reflow):

```css
.box {
  /* Translate — move */
  transform: translateX(50px);
  transform: translateY(-20px);
  transform: translate(50px, -20px);
  
  /* Scale — resize */
  transform: scale(1.5);       /* 150% size */
  transform: scaleX(2);
  transform: scale(0.5, 1.5);  /* half width, 150% height */
  
  /* Rotate */
  transform: rotate(45deg);
  transform: rotateX(45deg);   /* 3D rotation */
  transform: rotateY(45deg);
  
  /* Skew */
  transform: skewX(10deg);
  transform: skew(10deg, 5deg);
  
  /* Combine multiple transforms */
  transform: translateX(50px) rotate(45deg) scale(1.2);
  /* ⚠️ Order matters — applied right to left */
  
  /* Transform origin — default is center */
  transform-origin: top left;
  transform-origin: 50% 100%;   /* bottom center */
}
```

### Performance — what to animate

```css
/* CHEAP to animate (compositor only — no layout or paint): */
/* transform, opacity */

/* EXPENSIVE to animate (trigger layout recalculation): */
/* width, height, margin, padding, top, left, font-size */

/* BAD */
.box { transition: width 0.3s, left 0.3s; }

/* GOOD — achieve the same visual result with transform */
.box { transition: transform 0.3s; }
.box:hover { transform: translateX(20px) scale(1.1); }
```

### will-change

Hints to the browser that a property will change, allowing it to optimize in advance:

```css
.animated-element {
  will-change: transform, opacity;
  /* Browser promotes to its own compositor layer */
}
```

⚠️ Use sparingly — every `will-change` creates a new layer that consumes GPU memory. Apply only to elements that actually animate frequently. Remove after animation if applied via JS.

### Reduced motion

```css
/* Respect user's motion preferences */
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

# Pseudo-classes and Pseudo-elements

### Pseudo-classes — select elements by STATE

```css
/* User interaction */
a:hover { }             /* mouse over */
a:active { }            /* being clicked */
a:visited { }           /* already visited link */
button:focus { }        /* focused (click or Tab) */
button:focus-visible { }/* focused via KEYBOARD only — preferred for styling */
button:focus-within { } /* element or any descendant is focused */

/* Form states */
input:required { }
input:optional { }
input:valid { }
input:invalid { }
input:placeholder-shown { }  /* placeholder text is visible */
input:checked { }            /* checkbox/radio is checked */
input:disabled { }
input:enabled { }
input:read-only { }
input:autofill { }           /* browser autofilled this input */

/* Structural */
li:first-child { }
li:last-child { }
li:nth-child(3) { }          /* third item */
li:nth-child(odd) { }        /* 1st, 3rd, 5th... */
li:nth-child(even) { }       /* 2nd, 4th, 6th... */
li:nth-child(3n) { }         /* every 3rd: 3, 6, 9... */
li:nth-child(3n+1) { }       /* 1st of every 3: 1, 4, 7... */
li:nth-last-child(2) { }     /* 2nd from the end */
p:first-of-type { }          /* first <p> among siblings */
p:last-of-type { }
div:only-child { }           /* only child of its parent */
:root { }                    /* html element — for CSS variables */
:empty { }                   /* elements with no children or text */

/* Negation and matching */
:not(.active) { }            /* everything except .active */
:is(h1, h2, h3) { }         /* matches any — takes highest specificity */
:where(h1, h2, h3) { }      /* matches any — zero specificity */
:has(img) { }                /* parent selector */

/* Other */
:target { }                  /* element targeted by URL hash (#id) */
:lang(en) { }                /* elements with matching lang attribute */
```

### :focus vs :focus-visible

```css
/* :focus — fires on ANY focus (click, Tab, programmatic) */
button:focus {
  outline: 2px solid blue;
  /* Problem: shows outline on mouse click too, which looks odd */
}

/* :focus-visible — fires only on KEYBOARD focus */
button:focus-visible {
  outline: 2px solid blue;
  outline-offset: 2px;
  /* Only shows when user is navigating with Tab — clean UX */
}

/* Modern best practice */
button:focus {
  outline: none;  /* remove default for mouse clicks */
}
button:focus-visible {
  outline: 2px solid #4A90D9;
  outline-offset: 2px;  /* accessible keyboard indicator */
}
```

### :nth-child() advanced

```css
/* Select first 5 items */
li:nth-child(-n+5) { }

/* Select everything after the 3rd item */
li:nth-child(n+4) { }

/* Select items 3 through 7 */
li:nth-child(n+3):nth-child(-n+7) { }

/* Modern: nth-child with selector */
/* Select every odd <p> that has class .highlight */
p.highlight:nth-child(odd of .highlight) { }
```

### Pseudo-elements — style PARTS of an element

```css
/* ::before and ::after — insert content before/after element */
.quote::before {
  content: "\201C";  /* opening curly quote " */
  font-size: 2rem;
  color: gray;
}

.quote::after {
  content: "\201D";  /* closing curly quote " */
}

/* content is REQUIRED for ::before and ::after to appear */
/* Even for decorative elements: */
.divider::after {
  content: "";
  display: block;
  width: 100%;
  height: 1px;
  background: #ddd;
}

/* ::placeholder — style input placeholder text */
input::placeholder {
  color: #999;
  font-style: italic;
}

/* ::selection — style highlighted/selected text */
::selection {
  background: #4A90D9;
  color: white;
}

/* ::first-line and ::first-letter */
p::first-letter {
  font-size: 3rem;
  float: left;
  line-height: 1;
  margin-right: 8px;
}

p::first-line {
  font-weight: bold;
}

/* ::marker — style list bullets/numbers */
li::marker {
  color: #4A90D9;
  font-weight: bold;
}
```

### Practical pseudo-element patterns

```css
/* Required field indicator */
label.required::after {
  content: " *";
  color: red;
}

/* External link indicator */
a[href^="http"]::after {
  content: " ↗";
  font-size: 0.8em;
}

/* Overlay on images */
.image-container {
  position: relative;
}
.image-container::after {
  content: "";
  position: absolute;
  inset: 0;
  background: linear-gradient(transparent 50%, rgba(0,0,0,0.7));
}

/* Custom checkbox */
input[type="checkbox"] {
  appearance: none;
  width: 20px;
  height: 20px;
  border: 2px solid #ccc;
  border-radius: 4px;
}
input[type="checkbox"]:checked::after {
  content: "✓";
  display: block;
  text-align: center;
  color: #4A90D9;
  font-weight: bold;
}
```

# Typography

### Font loading

```css
/* System font stack — no loading needed */
body {
  font-family: system-ui, -apple-system, BlinkMacSystemFont, 
    'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
}

/* Custom font with @font-face */
@font-face {
  font-family: 'MyFont';
  src: url('/fonts/myfont.woff2') format('woff2'),
       url('/fonts/myfont.woff') format('woff');
  font-weight: 400;
  font-style: normal;
  font-display: swap;  /* show fallback immediately, swap when loaded */
}
```

### font-display

```css
@font-face {
  font-family: 'MyFont';
  src: url('myfont.woff2');
  
  font-display: auto;      /* browser decides (usually block) */
  font-display: block;     /* invisible text up to 3s, then swap (FOIT) */
  font-display: swap;      /* show fallback immediately, swap when ready (FOUT) */
  font-display: fallback;  /* very short invisible period, then fallback, may swap */
  font-display: optional;  /* show fallback, only use font if already cached */
}
```

**Recommended:** `swap` for body text (content is visible immediately), `optional` for non-critical fonts (prevents layout shift on slow connections).

### Line height and vertical rhythm

```css
body {
  line-height: 1.5;    /* unitless — multiplied by element's font-size */
  /* 16px font × 1.5 = 24px line height */
  /* Unitless is preferred — inherits the multiplier, not the computed value */
}

h1 {
  line-height: 1.2;    /* tighter for headings */
}

/* BAD — using px or em for line-height */
body {
  font-size: 16px;
  line-height: 24px;   /* doesn't scale with font-size changes */
}
```

### Text overflow

```css
/* Single line truncation */
.truncate {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  /* ⚠️ Needs a constrained width — won't work on inline elements */
}

/* Multi-line truncation (webkit only but widely supported) */
.truncate-multi {
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 3;  /* show 3 lines then ellipsis */
  overflow: hidden;
}

/* Word breaking */
.long-text {
  overflow-wrap: break-word;     /* break long words to prevent overflow */
  word-break: break-all;         /* break at any character (useful for URLs) */
  hyphens: auto;                 /* add hyphens when breaking — needs lang attribute */
}
```

### Other typography properties

```css
.text {
  /* Spacing */
  letter-spacing: 0.05em;    /* space between characters */
  word-spacing: 0.1em;       /* space between words */
  text-indent: 2em;          /* indent first line */
  
  /* Transform */
  text-transform: uppercase;
  text-transform: lowercase;
  text-transform: capitalize;
  
  /* Decoration */
  text-decoration: underline;
  text-decoration: line-through;
  text-decoration: underline wavy red;  /* style and color */
  text-underline-offset: 4px;           /* space between text and underline */
  
  /* Alignment */
  text-align: left;
  text-align: center;
  text-align: justify;
  
  /* Wrapping */
  white-space: normal;     /* default: wrap text */
  white-space: nowrap;     /* no wrapping */
  white-space: pre;        /* preserve whitespace and newlines */
  white-space: pre-wrap;   /* preserve whitespace, allow wrapping */
  
  /* Font features */
  font-variant-numeric: tabular-nums;   /* equal-width numbers for alignment */
  font-variant-numeric: oldstyle-nums;  /* numbers that blend with text */
  font-feature-settings: "kern" 1;      /* enable kerning */
}
```

# Overflow and Scrolling

### Overflow

```css
.container {
  overflow: visible;   /* default: content overflows the box */
  overflow: hidden;    /* clips content, no scrollbar */
  overflow: scroll;    /* always shows scrollbar */
  overflow: auto;      /* scrollbar only when content overflows */
  overflow: clip;      /* like hidden but doesn't create a scroll container */
  
  /* Individual axes */
  overflow-x: auto;
  overflow-y: hidden;
}
```

**`overflow: hidden` vs `overflow: clip`:**
- `hidden` creates a scroll container (can be scrolled via JS) and establishes a new BFC
- `clip` truly clips content — no scrolling possible, no BFC. More predictable and often what you actually want.

### Scroll behavior

```css
/* Smooth scrolling for anchor links */
html {
  scroll-behavior: smooth;
}

/* Respecting reduced motion preference */
@media (prefers-reduced-motion: reduce) {
  html {
    scroll-behavior: auto;
  }
}
```

```javascript
// Programmatic smooth scrolling
element.scrollIntoView({ behavior: 'smooth', block: 'start' });
```

### Scroll snap

Creates a "snap to" effect — content snaps to defined positions when scrolling:

```css
/* Container */
.carousel {
  display: flex;
  overflow-x: auto;
  scroll-snap-type: x mandatory;   /* snap horizontally, always snap */
  scroll-snap-type: y proximity;   /* snap vertically, snap if close enough */
  
  -webkit-overflow-scrolling: touch; /* smooth scrolling on iOS (legacy) */
}

/* Items */
.carousel-item {
  scroll-snap-align: start;   /* snap to the start of this item */
  scroll-snap-align: center;  /* snap to the center */
  scroll-snap-align: end;     /* snap to the end */
  
  flex: 0 0 100%;  /* each item takes full width */
}

/* Prevent scrolling past first/last item */
.carousel {
  scroll-snap-stop: always;  /* must stop at each item, can't skip */
}

/* Scroll padding — offset the snap position */
.carousel {
  scroll-padding: 0 20px;  /* 20px padding before snap position */
}
```

### Overscroll behavior

Controls what happens when you reach the edge of a scrollable area:

```css
.modal-body {
  overflow-y: auto;
  overscroll-behavior: contain;  /* prevents scroll from "leaking" to parent */
}

/* Values: */
/* auto — default: scroll chains to parent */
/* contain — prevent scroll chaining, keep bounce effect */
/* none — prevent scroll chaining AND bounce effect */

/* Prevent pull-to-refresh on mobile */
body {
  overscroll-behavior-y: none;
}
```

### Scrollbar styling

```css
/* Webkit browsers (Chrome, Safari, Edge) */
.container::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}

.container::-webkit-scrollbar-track {
  background: #f0f0f0;
  border-radius: 4px;
}

.container::-webkit-scrollbar-thumb {
  background: #888;
  border-radius: 4px;
}

.container::-webkit-scrollbar-thumb:hover {
  background: #555;
}

/* Modern standard (limited support but growing) */
.container {
  scrollbar-width: thin;         /* auto | thin | none */
  scrollbar-color: #888 #f0f0f0; /* thumb-color track-color */
}

/* Hide scrollbar but keep scrolling */
.container {
  scrollbar-width: none;
}
.container::-webkit-scrollbar {
  display: none;
}
```

# CSS Functions

### calc()

```css
.sidebar {
  width: calc(100% - 250px);      /* remaining space after 250px sidebar */
  height: calc(100vh - 60px);     /* full height minus header */
  padding: calc(1rem + 2vw);      /* mix fixed and responsive */
  font-size: calc(14px + 0.5vw);  /* responsive font */
}

/* Can nest and mix units */
.box {
  width: calc(100% - calc(2 * var(--spacing)));
  /* same as: */ width: calc(100% - 2 * var(--spacing));
}

/* ⚠️ Spaces around operators are REQUIRED */
/* calc(100%-20px)  ← WRONG */
/* calc(100% - 20px) ← CORRECT */
```

### min(), max(), clamp()

```css
.container {
  /* min() — picks the SMALLER value */
  width: min(90vw, 1200px);
  /* If viewport is 1000px: min(900px, 1200px) = 900px */
  /* If viewport is 2000px: min(1800px, 1200px) = 1200px */
  
  /* max() — picks the LARGER value */
  width: max(300px, 50vw);
  /* Ensures minimum width of 300px */
  
  /* clamp(min, preferred, max) — value with boundaries */
  width: clamp(300px, 90vw, 1200px);
  /* Same as: max(300px, min(90vw, 1200px)) */
  
  font-size: clamp(1rem, 2.5vw, 2rem);
  /* Font scales with viewport but stays between 1rem and 2rem */
  
  padding: clamp(1rem, 3vw, 3rem);
}
```

### var()

```css
:root {
  --spacing: 16px;
  --color: #4A90D9;
}

.box {
  padding: var(--spacing);
  color: var(--color);
  
  /* With fallback */
  margin: var(--margin, 20px);  /* uses 20px if --margin isn't defined */
  
  /* Nested fallback */
  color: var(--theme-color, var(--color, black));
}
```

### env()

Accesses environment variables — mainly used for safe areas on notched devices:

```css
.content {
  padding-top: env(safe-area-inset-top);
  padding-bottom: env(safe-area-inset-bottom);
  padding-left: env(safe-area-inset-left);
  padding-right: env(safe-area-inset-right);
  
  /* With fallback */
  padding-top: env(safe-area-inset-top, 20px);
}
```

Required for iPhones with notch/dynamic island to prevent content from going under the notch.

Also needs this meta tag:
```html
<meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover" />
```

### Other functions

```css
.element {
  /* Color functions */
  background: color-mix(in srgb, #ff0000, #0000ff 50%);  /* mix two colors */
  background: rgb(from var(--color) r g b / 50%);         /* derive with alpha */
  
  /* Shape functions */
  clip-path: circle(50%);
  clip-path: polygon(0 0, 100% 0, 100% 100%);  /* triangle */
  clip-path: inset(10px 20px round 8px);
  
  /* Gradient functions */
  background: linear-gradient(135deg, #ff0000, #0000ff);
  background: radial-gradient(circle at center, #ff0000, transparent);
  background: conic-gradient(from 0deg, red, yellow, green, blue, red);
  
  /* Filter functions */
  filter: blur(4px);
  filter: brightness(1.2);
  filter: contrast(1.5);
  filter: grayscale(100%);
  filter: drop-shadow(2px 4px 6px rgba(0,0,0,0.3));
  backdrop-filter: blur(10px);  /* blurs content BEHIND the element */
}
```

# Modern CSS Features

### Cascade Layers (@layer)

Cascade layers give you explicit control over which styles take priority, regardless of specificity or source order:

```css
/* Define layer order — first layer has LOWEST priority */
@layer reset, base, components, utilities;

@layer reset {
  * { margin: 0; padding: 0; }
}

@layer base {
  a { color: blue; }  /* specificity: 0-0-1 */
}

@layer components {
  .nav a { color: white; }  /* specificity: 0-1-1 */
}

@layer utilities {
  .text-red { color: red !important; }
}

/* Layer order determines winner, not specificity */
/* utilities > components > base > reset */
/* Unlayered styles beat ALL layers */
```

This solves the specificity wars that happen in large codebases. Put third-party CSS in a low-priority layer, and your styles always win without needing `!important`.

### CSS Nesting

```css
/* Native CSS nesting — no preprocessor needed */
.card {
  padding: 16px;
  
  .title {
    font-size: 1.5rem;
    
    &:hover {
      color: blue;
    }
  }
  
  .body {
    line-height: 1.6;
  }
  
  /* & is required for pseudo-classes and combinators */
  &:hover {
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  }
  
  &.featured {
    border-color: gold;
  }
  
  /* Media queries can be nested too */
  @media (max-width: 768px) {
    padding: 8px;
  }
}
```

### @scope

Scopes styles to a specific DOM subtree:

```css
@scope (.card) {
  /* These styles ONLY apply inside .card */
  h2 { font-size: 1.5rem; }
  p { color: gray; }
}

/* With a lower boundary — style between two selectors */
@scope (.card) to (.card-footer) {
  /* Styles apply inside .card but NOT inside .card-footer */
  p { color: gray; }
}
```

### color-mix()

```css
.button {
  --base-color: #4A90D9;
  background: var(--base-color);
}

.button:hover {
  /* Darken by mixing with black */
  background: color-mix(in srgb, var(--base-color), black 20%);
}

.button:active {
  /* Darken more */
  background: color-mix(in srgb, var(--base-color), black 40%);
}

.button.light {
  /* Lighten by mixing with white */
  background: color-mix(in srgb, var(--base-color), white 30%);
}
```

### @starting-style

Enables transitions from `display: none`:

```css
.modal {
  display: none;
  opacity: 0;
  transform: translateY(-20px);
  transition: opacity 0.3s, transform 0.3s, display 0.3s allow-discrete;
  
  @starting-style {
    opacity: 0;
    transform: translateY(-20px);
  }
}

.modal.open {
  display: block;
  opacity: 1;
  transform: translateY(0);
}
```

### View transitions

```css
/* Simple page transition */
::view-transition-old(root) {
  animation: fade-out 0.3s ease-out;
}

::view-transition-new(root) {
  animation: fade-in 0.3s ease-in;
}
```

```javascript
// Trigger a view transition
document.startViewTransition(() => {
  // Update the DOM
  updateContent();
});
```

# Performance

### Repaint vs Reflow

**Reflow (Layout)** — recalculates positions and sizes of elements. Expensive.
Triggered by: changing width, height, margin, padding, font-size, adding/removing elements, reading layout properties (offsetHeight, scrollTop).

**Repaint** — redraws pixels without changing layout. Cheaper.
Triggered by: changing color, background, visibility, box-shadow.

**Composite** — cheapest. Only moves layers around.
Triggered by: transform, opacity (when on their own compositor layer).

```
Most expensive ←————————————→ Least expensive
Reflow (Layout) → Repaint → Composite
```

### Triggering forced reflow

```javascript
// BAD — forces reflow in every iteration (layout thrashing)
for (const el of elements) {
  el.style.width = container.offsetWidth + 'px'; // read forces reflow
}

// GOOD — batch reads, then batch writes
const width = container.offsetWidth; // single read
for (const el of elements) {
  el.style.width = width + 'px'; // all writes together
}
```

### contain

Tells the browser that an element's internals are independent from the rest of the page:

```css
.widget {
  contain: layout;    /* layout changes inside don't affect outside */
  contain: paint;     /* content doesn't paint outside this box */
  contain: size;      /* element's size doesn't depend on children */
  contain: style;     /* counters/quotes inside don't affect outside */
  contain: content;   /* shorthand for layout + paint */
  contain: strict;    /* shorthand for layout + paint + size */
}

/* Use case: a card component that shouldn't affect siblings */
.card {
  contain: content;
  /* Browser can optimize — changes inside won't trigger reflow outside */
}
```

### content-visibility

```css
/* Skips rendering of off-screen content */
.section {
  content-visibility: auto;
  contain-intrinsic-size: 0 500px; /* estimated height to prevent layout shift */
}

/* Browser skips layout, paint, and style for off-screen sections */
/* Massive performance win for long pages with many sections */
```

### will-change best practices

```css
/* DO: apply before animation starts */
.card:hover {
  will-change: transform;
}
.card:hover .inner {
  transform: scale(1.05);
}

/* DON'T: apply globally to everything */
* { will-change: transform, opacity; }  /* wastes GPU memory */

/* DON'T: apply permanently */
.card { will-change: transform; }  /* should be temporary */
```

### Critical CSS pattern

```html
<head>
  <!-- Inline critical CSS — no external request needed -->
  <style>
    /* Only above-the-fold styles */
    body { margin: 0; font-family: system-ui; }
    .hero { height: 100vh; display: flex; align-items: center; }
    .nav { height: 60px; display: flex; }
  </style>
  
  <!-- Load full CSS without blocking render -->
  <link rel="preload" href="styles.css" as="style" 
    onload="this.onload=null;this.rel='stylesheet'" />
  <noscript>
    <link rel="stylesheet" href="styles.css" />
  </noscript>
</head>
```

### CSS containment for performance checklist

```
Technique                    | Impact
-----------------------------|---------------------------
Animate only transform/opacity| Avoids reflow and repaint
Use contain: content         | Isolates reflow
content-visibility: auto     | Skips off-screen rendering
will-change (sparingly)      | Pre-promotes to GPU layer
Reduce DOM depth             | Faster style calculation
Avoid layout thrashing       | Batch reads and writes
Critical CSS inlining        | Faster first paint
```

# BEM and Architecture

### BEM (Block Element Modifier)

A naming convention that makes CSS predictable and maintainable:

```
Block:    .card
Element:  .card__title       (part of the block)
Modifier: .card--featured    (variation of the block)
```

```css
/* Block */
.card {
  padding: 16px;
  border: 1px solid #ddd;
  border-radius: 8px;
}

/* Element — part of the block (double underscore) */
.card__title {
  font-size: 1.25rem;
  font-weight: bold;
}

.card__body {
  color: #666;
  line-height: 1.6;
}

.card__image {
  width: 100%;
  border-radius: 8px 8px 0 0;
}

/* Modifier — variation (double hyphen) */
.card--featured {
  border-color: gold;
  background: #fffdf0;
}

.card--compact {
  padding: 8px;
}

/* Element with modifier */
.card__title--large {
  font-size: 2rem;
}
```

```html
<article class="card card--featured">
  <img class="card__image" src="..." alt="..." />
  <h2 class="card__title card__title--large">Featured Post</h2>
  <p class="card__body">Content here...</p>
</article>
```

### Why BEM works

```css
/* Without BEM — specificity conflicts, unclear relationships */
.card { }
.card .title { }           /* what if there's a nested component with .title? */
.card.featured .title { }  /* specificity keeps climbing */
.sidebar .card .title { }  /* location-dependent — fragile */

/* With BEM — flat specificity, clear ownership */
.card { }
.card__title { }           /* clearly belongs to card */
.card--featured { }        /* clearly a variation of card */
/* Every selector is one class — specificity is always 0-1-0 */
```

### CSS architecture approaches

**Utility-first (Tailwind CSS):**
```html
<article class="p-4 border border-gray-200 rounded-lg bg-yellow-50">
  <h2 class="text-xl font-bold">Featured Post</h2>
  <p class="text-gray-600 leading-relaxed">Content...</p>
</article>
```

Pros: no naming decisions, no unused CSS, rapid prototyping.
Cons: verbose HTML, harder to read, design consistency requires a system.

**CSS-in-JS (styled-components, Emotion):**
```javascript
const Card = styled.article`
  padding: 16px;
  border: 1px solid #ddd;
  border-radius: 8px;
  
  ${props => props.featured && css`
    border-color: gold;
    background: #fffdf0;
  `}
`;
```

Pros: scoped styles by default, dynamic styles with props, co-located with components.
Cons: runtime cost, larger bundle, can't cache CSS separately.

**CSS Modules:**
```css
/* Card.module.css */
.card { padding: 16px; }
.title { font-size: 1.25rem; }
.featured { border-color: gold; }
```

```javascript
import styles from './Card.module.css';
<article className={`${styles.card} ${styles.featured}`}>
  <h2 className={styles.title}>Post</h2>
</article>
```

Pros: scoped by default (classes are hashed), standard CSS syntax, no runtime cost.
Cons: class composition can be verbose, harder to share styles.

### Choosing an approach

```
Approach      | Best for                          | Avoid when
------------- |-----------------------------------|----------------------------
BEM           | Large teams, long-lived projects   | Small projects, prototypes
Tailwind      | Rapid development, small teams     | Complex components, readability priority
CSS-in-JS     | React apps, dynamic theming        | Performance-critical, SSR complexity
CSS Modules   | Component-based apps               | Need for global utilities
@layer        | Mixing third-party + custom CSS    | Simple projects
```

In practice, most modern React projects use either Tailwind or CSS Modules. BEM is still valuable for non-framework projects and for understanding CSS architecture principles.
