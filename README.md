# CSS Concepts — Fixes and Additions

## Fix 1: Add language tags to ALL code blocks

This is a bulk operation. In your code editor, do a find-and-replace:

**Find:** ` ```\n` (triple backtick followed by newline)

For CSS code blocks (the majority), replace with ` ```css\n`
For HTML snippets, replace with ` ```html\n`
For JavaScript snippets, replace with ` ```javascript\n`
For plain text tables/diagrams, leave as ` ``` `

You have ~80 code blocks that need this. Most are CSS. Do it section by section — it takes 10 minutes.

---

## Fix 2: Display and Layout — fill in missing "display: none vs visibility: hidden"

Replace the empty `### display: none vs visibility: hidden` heading with:

```css
/* display: none — completely removed from layout */
.hidden {
  display: none;
  /* No space taken, not accessible, not interactive */
  /* Children cannot override — everything inside is hidden */
}

/* visibility: hidden — invisible but space preserved */
.invisible {
  visibility: hidden;
  /* Space still taken, not accessible, not interactive */
  /* Children CAN override with visibility: visible */
}
```

```html
<!-- visibility: hidden — child can make itself visible again -->
<div style="visibility: hidden;">
  I'm invisible
  <span style="visibility: visible;">But I'm visible!</span>
</div>

<!-- display: none — child cannot escape -->
<div style="display: none;">
  I'm gone
  <span style="display: block;">I'm still gone — parent removes me from the tree</span>
</div>
```

This matters in practice for things like dropdown menus and tooltips — you might want the container invisible but a child element visible on hover. `visibility` allows this, `display: none` doesn't.

Also consider `opacity: 0` — the element is invisible but still takes space, is still accessible to screen readers, and is still interactive (clickable). Use it for fade animations where you need the element to remain in the layout.

---

## Fix 3: Units — fill in missing viewport units code examples

Replace the viewport units section with this complete version:

```css
.hero {
  /* Basic viewport units */
  height: 100vh;       /* 100% of viewport height */
  width: 100vw;        /* 100% of viewport width */
  font-size: 5vmin;    /* 5% of the smaller dimension (width or height) */
  font-size: 5vmax;    /* 5% of the larger dimension */

  /* New viewport units (solve the mobile address bar problem) */
  height: 100dvh;      /* dynamic: updates when address bar shows/hides */
  height: 100svh;      /* small: viewport WITH address bar visible (smallest) */
  height: 100lvh;      /* large: viewport WITHOUT address bar (largest) */
}
```

**The mobile viewport problem:** on mobile browsers, `100vh` equals the large viewport height (address bar hidden). But when the page first loads, the address bar is visible, making the actual viewport smaller. Content set to `100vh` overflows, causing a scrollbar on what should be a full-screen section.

```css
/* OLD — broken on mobile */
.hero {
  height: 100vh;  /* overflows when address bar is visible */
}

/* NEW — works correctly on mobile */
.hero {
  height: 100dvh;  /* adjusts dynamically as address bar appears/disappears */
}

/* SAFE FALLBACK — for browsers that don't support dvh */
.hero {
  height: 100vh;    /* fallback */
  height: 100dvh;   /* override for supporting browsers */
}
```

`dvh` changes value as the address bar animates in/out. `svh` is the smallest possible viewport (bar visible). `lvh` is the largest (bar hidden). For most use cases, `dvh` is what you want.

---

## Addition 1: Logical Properties

Add this as a new section after "Units" (or at the end before BEM):

---

# Logical Properties

Traditional CSS uses physical directions: top, right, bottom, left. Logical properties use flow-relative directions: block (vertical in English) and inline (horizontal in English). This matters for internationalization — in RTL (right-to-left) languages like Arabic, "start" means right, not left.

```css
/* Physical (traditional) → Logical (modern) */

/* Margin */
margin-top    → margin-block-start
margin-bottom → margin-block-end
margin-left   → margin-inline-start
margin-right  → margin-inline-end
margin: 10px 20px → margin-block: 10px; margin-inline: 20px;

/* Padding — same pattern */
padding-top    → padding-block-start
padding-bottom → padding-block-end
padding-left   → padding-inline-start
padding-right  → padding-inline-end

/* Size */
width      → inline-size
height     → block-size
min-width  → min-inline-size
max-height → max-block-size

/* Position */
top    → inset-block-start
bottom → inset-block-end
left   → inset-inline-start
right  → inset-inline-end
/* Shorthand: */
top: 0; right: 0; bottom: 0; left: 0;  →  inset: 0;

/* Border */
border-top    → border-block-start
border-left   → border-inline-start
border-radius: 8px 0 0 8px → border-start-start-radius: 8px; border-end-start-radius: 8px;

/* Text */
text-align: left  → text-align: start
text-align: right → text-align: end
```

### Practical examples

```css
/* Centering — logical way */
.container {
  max-inline-size: 1200px;  /* max-width */
  margin-inline: auto;       /* margin-left + margin-right: auto */
}

/* Card with directional spacing */
.card {
  padding-block: 16px;      /* top and bottom */
  padding-inline: 24px;     /* left and right (or right and left in RTL) */
  border-inline-start: 4px solid blue;  /* left border in LTR, right in RTL */
}

/* Absolute positioning shorthand */
.overlay {
  position: absolute;
  inset: 0;  /* top: 0; right: 0; bottom: 0; left: 0; */
}

.sidebar {
  position: fixed;
  inset-block: 0;           /* top: 0; bottom: 0; */
  inset-inline-start: 0;    /* left: 0 in LTR, right: 0 in RTL */
  inline-size: 250px;       /* width: 250px */
}
```

### Why adopt logical properties

Even if you're not building RTL sites right now, logical properties are worth adopting because `margin-inline: auto` is cleaner than `margin-left: auto; margin-right: auto`, `inset: 0` replaces four separate properties, they future-proof your code for internationalization, and they're supported in all modern browsers.

---

## Addition 2: Aspect Ratio

Add this after Logical Properties or within the Units section:

---

# Aspect Ratio

Before `aspect-ratio`, maintaining proportions required the padding-top hack. Now it's one property:

```css
/* Modern — clean and declarative */
.video-container {
  aspect-ratio: 16 / 9;
  width: 100%;
  /* height is automatically calculated to maintain 16:9 ratio */
}

.square {
  aspect-ratio: 1;  /* shorthand for 1 / 1 */
  width: 200px;
  /* height automatically becomes 200px */
}

.portrait {
  aspect-ratio: 3 / 4;
}

/* The old padding-top hack (don't use anymore) */
.video-container-old {
  position: relative;
  width: 100%;
  padding-top: 56.25%;  /* 9/16 = 0.5625 = 56.25% */
}
.video-container-old iframe {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
}
```

### How aspect-ratio interacts with content

```css
.box {
  aspect-ratio: 2 / 1;
  width: 300px;
  /* height = 150px */
}

/* If content is taller than the aspect ratio allows, the box grows */
/* To prevent this, add overflow or min-height constraints */
.box {
  aspect-ratio: 2 / 1;
  width: 300px;
  overflow: hidden;  /* clip content that exceeds the ratio */
}
```

### Common aspect ratios

```
Ratio   | Use case
--------|---------------------------
1 / 1   | Profile pictures, icons, thumbnails
16 / 9  | Video embeds, hero images, widescreen content
4 / 3   | Classic photo format, older video
3 / 2   | DSLR photo format
21 / 9  | Ultra-wide banners
```

### Responsive images with aspect-ratio

```css
/* Prevent layout shift by setting aspect ratio matching the image dimensions */
img {
  width: 100%;
  height: auto;
  aspect-ratio: attr(width) / attr(height);  /* experimental — use width/height HTML attributes instead */
}

/* In practice, use HTML width and height attributes */
/* The browser calculates aspect ratio from them automatically */
```

```html
<img src="photo.jpg" width="800" height="600" alt="..." style="width: 100%; height: auto;" />
<!-- Browser knows the ratio is 4:3 before the image loads — no layout shift -->
```

---

## Addition 3: @supports — Feature Detection

Add to the Modern CSS Features section:

### @supports — feature detection in CSS

```css
/* Check if a property is supported before using it */
@supports (aspect-ratio: 1) {
  .card-image {
    aspect-ratio: 16 / 9;
  }
}

/* Fallback for browsers that don't support it */
@supports not (aspect-ratio: 1) {
  .card-image {
    padding-top: 56.25%;  /* old hack */
  }
}

/* Check for specific values */
@supports (display: grid) {
  .layout { display: grid; }
}

/* Combine conditions */
@supports (display: grid) and (gap: 1rem) {
  .grid { display: grid; gap: 1rem; }
}

/* Check for selector support */
@supports selector(:has(*)) {
  /* :has() is supported — use it */
  .card:has(img) { padding: 0; }
}

@supports not selector(:has(*)) {
  /* fallback for browsers without :has() */
  .card--with-image { padding: 0; }
}
```

`@supports` is CSS's native feature detection — like JavaScript's `if ('IntersectionObserver' in window)` but for CSS properties. Use it for progressive enhancement: modern browsers get the modern layout, older ones get the fallback.

---

## Addition 4: accent-color and color-scheme

Add to the Colors and Custom Properties section:

### accent-color — style form controls with one property

```css
/* Before accent-color, styling checkboxes/radios required 
   hiding the native control and building a custom one from scratch */

/* Now — one line changes the color of native form controls */
:root {
  accent-color: #4A90D9;
}

/* Applies to: checkboxes, radio buttons, range sliders, progress bars */
input[type="checkbox"] {
  accent-color: #4A90D9;  /* checkbox fill color when checked */
}

input[type="radio"] {
  accent-color: #e74c3c;  /* radio dot color */
}

input[type="range"] {
  accent-color: #27ae60;  /* slider track fill + thumb color */
}

progress {
  accent-color: #f39c12;  /* progress bar fill color */
}
```

The browser automatically handles contrast — if the accent color is dark, it makes the checkmark white, and vice versa. No custom CSS needed for the foreground.

### color-scheme — declare dark/light mode support

```css
/* Tell the browser your page supports both color schemes */
:root {
  color-scheme: light dark;
}

/* This does several things automatically:
   - Browser chrome (scrollbars, form controls) adapts to the user's preference
   - Default colors for text and backgrounds switch appropriately
   - form controls (inputs, selects, buttons) get dark-mode styling for free */
```

```css
/* Force a specific scheme on an element */
.always-dark {
  color-scheme: dark;
  /* This element's form controls and scrollbars will always be dark-themed */
}

.always-light {
  color-scheme: light;
}
```

```html
<!-- Can also be set via meta tag for instant application (no FOUC) -->
<meta name="color-scheme" content="light dark" />
```

The difference between `color-scheme` and `prefers-color-scheme`:
- `color-scheme` tells the browser what your page supports — it adapts native UI elements automatically
- `prefers-color-scheme` media query lets you write custom styles for each mode

Use both together:

```css
:root {
  color-scheme: light dark;  /* native controls adapt automatically */
  
  --bg: #ffffff;
  --text: #333333;
}

@media (prefers-color-scheme: dark) {
  :root {
    --bg: #1a1a2e;  /* your custom dark mode colors */
    --text: #e0e0e0;
  }
}
```

---

## Addition 5: Gradients — expanded coverage

Add to the CSS Functions section under "Other functions" (or as its own sub-section):

### Gradients in depth

```css
/* Linear gradient */
.box {
  /* Direction + color stops */
  background: linear-gradient(to right, #ff0000, #0000ff);
  background: linear-gradient(135deg, #ff0000, #0000ff);
  background: linear-gradient(to bottom right, #ff0000, #0000ff);
  
  /* Multiple color stops */
  background: linear-gradient(90deg, red, orange, yellow, green, blue);
  
  /* Control stop positions */
  background: linear-gradient(90deg, red 0%, red 50%, blue 50%, blue 100%);
  /* Hard stop at 50% — no gradient, just two solid halves */
  
  /* Repeating */
  background: repeating-linear-gradient(45deg, #f0f0f0, #f0f0f0 10px, #ffffff 10px, #ffffff 20px);
  /* Creates diagonal stripes */
}

/* Radial gradient */
.circle {
  background: radial-gradient(circle at center, #ff0000, transparent);
  background: radial-gradient(ellipse at top left, #ff0000, #0000ff);
  background: radial-gradient(circle at 30% 70%, #ff0000 0%, transparent 70%);
}

/* Conic gradient */
.pie-chart {
  background: conic-gradient(
    red 0deg 90deg,       /* 0-90 degrees: red (25%) */
    blue 90deg 180deg,    /* 90-180: blue (25%) */
    green 180deg 270deg,  /* 180-270: green (25%) */
    yellow 270deg 360deg  /* 270-360: yellow (25%) */
  );
  border-radius: 50%;  /* make it circular */
}

/* Practical: gradient text */
.gradient-text {
  background: linear-gradient(135deg, #667eea, #764ba2);
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  color: transparent;  /* fallback for non-webkit */
}

/* Practical: gradient border */
.gradient-border {
  border: 3px solid transparent;
  background-image: linear-gradient(white, white), linear-gradient(135deg, #667eea, #764ba2);
  background-origin: border-box;
  background-clip: padding-box, border-box;
}

/* Practical: overlay on image */
.hero {
  background: 
    linear-gradient(rgba(0,0,0,0.5), rgba(0,0,0,0.7)),
    url('hero.jpg') center/cover;
  /* Semi-transparent gradient over background image */
}
```

---

## Addition 6: Specificity — add !important alternatives

Add to the Selectors and Specificity section:

### Alternatives to !important

```css
/* Instead of !important, increase specificity deliberately */

/* 1. Double the class selector */
.button.button {
  color: red;  /* specificity: 0-2-0, beats a single .button */
}

/* 2. Use :where() for base styles (zero specificity — easy to override) */
:where(.button) {
  color: blue;  /* specificity: 0-0-0 */
}
.button {
  color: red;   /* specificity: 0-1-0, easily wins */
}

/* 3. Use @layer for third-party CSS (always loses to unlayered styles) */
@layer vendor {
  .button { color: blue; }  /* low priority — layered */
}
.button { color: red; }     /* wins — unlayered */

/* 4. Use inline styles via JS as a last resort (specificity: 1-0-0-0) */
element.style.color = 'red';  /* beats any selector-based rule */
```
