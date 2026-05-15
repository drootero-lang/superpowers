# Mobile Layout Standards

Reusable responsive design standard for cinematic and editorial marketing websites.
Apply this methodology to all future projects. Update as patterns evolve.

---

## Core Philosophy

Desktop and mobile are intentionally different experiences. A good mobile layout is
not a compressed desktop — it is a considered rearrangement that serves the same
narrative goals through a different physical context.

- **Prioritise readability, visual flow, and tap usability** over preserving desktop
  compositions. A two-column editorial split that works beautifully at 1280px will
  often need to become a single intentional column at 390px.
- **Editorial and cinematic sections should feel intentional on mobile, not merely
  stacked.** Spacing, image crops, and type sizing should be actively tuned for
  narrow viewports, not just inherited from desktop values.
- **Apply mobile overrides in a clearly labelled block at the end of each CSS file**,
  separated from desktop rules. This makes QA, handoff, and future edits predictable.
- **Never change desktop rules to fix a mobile problem.** Use `@media` blocks scoped
  to the target breakpoint.

---

## Required Responsive Test Widths

Always test at the following CSS viewport widths before marking a page complete:

| Width | Device reference |
|-------|-----------------|
| 375px | iPhone SE, older iPhones |
| 390px | iPhone 14 / 15 Pro |
| 414px | iPhone 11, XR, XS Max |
| 430px | iPhone 14 Plus / 15 Plus |
| 768px | iPad portrait |
| 1024px | iPad landscape, small laptops |

Test in both portrait orientation and, where relevant, landscape. Pay particular
attention to 375px — it is the smallest common viewport and will expose any sizing
or overflow issues that wider phones hide.

---

## Hero Section Standards

### Viewport Height

**Never rely on `100vh` alone for full-viewport hero sections.**

On iOS Safari, `100vh` is calculated including the browser chrome (address bar and
toolbar), which means the hero extends behind the UI and content near the bottom is
obscured or requires scrolling to reach. Use `100svh` (smallest viewport height —
the stable visible area) instead, with a `min-height` floor for safety.

```css
/* Correct mobile hero height */
@media (max-width: 768px) {
  .hero {
    height: 100svh;
    min-height: 560px;
  }
}
```

`100svh` is supported in all modern browsers as of 2023. Provide `min-height` as a
floor for very short/landscape viewports.

### Navigation Height Correction

Hero sections that use `calc(var(--nav-height) + ...)` for their top padding must
switch to the mobile nav height token on small screens. Desktop navs are typically
taller (80–90px) than mobile navs (56–68px). Using the desktop value on mobile
wastes 20–30px of precious above-the-fold space.

```css
:root {
  --nav-height:     88px; /* desktop */
  --nav-height-mob: 64px; /* mobile ≤900px */
}

@media (max-width: 768px) {
  .hero__content {
    padding-top: calc(var(--nav-height-mob) + var(--space-8));
  }
}
```

### Headline Wrapping

Headlines must **wrap intentionally** — never mid-word, never leaving a single word
orphaned on the last line if it reads poorly.

- Use `font-size` and `max-width` together to control where line breaks fall.
- `white-space: nowrap` forces a single line when the font size is small enough
  to fit the full string; use this when one line is the correct design intent.
- When two intentional lines are required, set a `max-width` that fits the first
  line grouping but not the full string.
- **If the JS-powered letter-reveal splits each character into its own
  `display: inline-block` span**, word-boundary wrapping does not apply — the
  browser can break between any two spans. Solutions:
  - Reduce `max-width` so each intended line group fits without spilling.
  - Target the space span at the desired break point by `nth-child()` position and
    set `width: 100%; height: 0; overflow: hidden` to force a line break there.
  - Reduce font size until the full string fits on one clean line.

### CTA Buttons

- At ≤640px, stacked hero CTAs should go full-width or near-full-width.
- Minimum tap target: 44×44px (WCAG 2.5.5).
- Use `flex-direction: column` on the button group and `width: 100%` on each button.

```css
@media (max-width: 640px) {
  .hero__actions {
    flex-direction: column;
    width: 100%;
    max-width: 320px;
    margin-inline: auto;
  }
  .hero__actions .btn {
    width: 100%;
    justify-content: center;
  }
}
```

### Logo Sizing

On mobile, hero logos should be large enough to establish brand identity but not so
large they push CTA buttons below the fold. Use `clamp()` to scale with viewport
width and cap the maximum.

### Cinematic Overlays

Gradient overlays and glow effects that look atmospheric at desktop can read as dirty
or heavy on small screens. Audit overlay opacity values specifically at 390px.

---

## Editorial Split Section Standards

### Image/Text Order Rule

**On mobile and tablet stacked layouts, images should generally appear above text.**

Readers need visual context before they can engage with supporting copy. On desktop,
alternating image/text positions creates rhythm; on mobile, that rhythm collapses
and the order of content becomes the reader's guide.

**Default stacked order:**
1. Image (establishes the subject visually)
2. Heading / eyebrow
3. Body copy
4. CTA

Only deviate from this order when the section clearly works better the other way
(e.g., a section where the copy is the emotional hook and the image is supplementary).

### The Critical `order` Rule — Read This Before Using It

**`order` only affects direct children of a flex or grid container.**

This is the most common source of broken mobile layouts in editorial splits. Before
applying `order`, identify the **actual grid/flex items** — the direct children of
the container — not nested descendants.

#### Example of the wrong approach

```html
<div class="section__grid">                 <!-- grid container -->
  <div class="section__content">...</div>   <!-- grid item 1 -->
  <div>                                     <!-- grid item 2 (anonymous, no class) -->
    <div class="section__image-wrap">...</div>  <!-- NOT a grid item -->
  </div>
</div>
```

```css
/* WRONG — .section__image-wrap is a grandchild, not a grid item.
   This rule has no effect on grid stacking order. */
@media (max-width: 900px) {
  .section__image-wrap {
    order: -1;
  }
}
```

When the grid collapses to one column, both grid items (`section__content` and the
anonymous `<div>`) default to `order: 0`. DOM order wins: content appears first,
image appears below — the opposite of what is intended.

#### Correct approach A — Order the sibling content block

Push the content column down by giving it a higher `order` value. The image wrapper
stays at `order: 0` (default) and floats to the top.

```css
@media (max-width: 900px) {
  .section__grid {
    grid-template-columns: 1fr;
  }

  /* Move content below the image wrapper */
  .section__content {
    order: 1;
  }
  /* Image wrapper div (even without a class) stays at order: 0 — appears first */
}
```

#### Correct approach B — Order the direct grid child (when it has a class)

If the image column has its own class as a direct grid child, apply `order` there.

```css
@media (max-width: 900px) {
  .section__image-col {
    order: -1;
  }
  /* content stays at order: 0 and follows */
}
```

#### Correct approach C — Image is in a flex column, not the grid

When the image lives inside a flex column alongside text (common in intro columns
where copy and image share a single grid cell), use `order` within that flex context.

```html
<div class="section__intro">   <!-- flex column -->
  <h2>Heading</h2>
  <p>Body copy...</p>
  <div class="section__image-wrap">...</div>  <!-- flex child — order works here -->
</div>
```

```css
@media (max-width: 900px) {
  .section__image-wrap {
    order: -1;  /* moves image above heading and copy within the flex column */
    margin-top: 0;
  }
}
```

### Image Aspect Ratios on Mobile

Portrait images (e.g., `aspect-ratio: 4/5` or `3/4`) at full container width on
mobile produce very tall frames that dominate the page and force excessive scrolling.
Switch to landscape ratios when stacked.

```css
/* Desktop: tall portrait crop for editorial weight */
.section__image-wrap {
  aspect-ratio: 4 / 5;
}

/* Mobile: landscape crop for stacked single-column layout */
@media (max-width: 600px) {
  .section__image-wrap {
    aspect-ratio: 4 / 3;
  }
}

/* Tablet: widescreen crop when collapsing from two columns */
@media (max-width: 900px) {
  .section__image-wrap {
    aspect-ratio: 16 / 9;
  }
}
```

### Text Width Constraints

`max-width` values set in `ch` units or fixed `px` on body copy columns often exceed
the available width on narrow phones. Always remove or widen these constraints when
going single-column.

```css
@media (max-width: 900px) {
  .section__body {
    max-width: unset;
  }
}
```

---

## Section Spacing Standards

Large `padding-block` values (80px+) that create breathing room on desktop become
wasteful on mobile, where every pixel of vertical space is scrolled. Audit and
reduce internal padding at mobile breakpoints.

### Closing / CTA Sections

Full-bleed cinematic closing sections frequently use `padding: 5rem gutter`
(80px top/bottom). Halve this on mobile.

```css
@media (max-width: 640px) {
  .section-close__content {
    padding-block: 3rem; /* was 5rem */
  }
}
```

### Pull Quote Gap Above Content

Large `margin-bottom` values on intro pull quotes (48px–64px) feel right at desktop
but create dead space on mobile when everything is single-column. Reduce to ~40px.

```css
@media (max-width: 540px) {
  .section__quote {
    margin-bottom: var(--space-10); /* 40px vs 64px desktop */
  }
}
```

### Panel Visual Heights

Atmospheric CSS scene panels with tall portrait `aspect-ratio` values (e.g., `3/4`)
stack into very long pages at single-column. Switch to `16/9` when stacked.

```css
@media (max-width: 540px) {
  .moment__visual {
    aspect-ratio: 16 / 9;
  }
}
```

---

## Design Token Hygiene

### Always define the full spacing scale

Gaps in the spacing token scale (e.g., defining `--space-6` and `--space-8` but not
`--space-7`) cause silent failures. When a component references an undefined custom
property, the entire declaration becomes invalid and the property falls back to its
initial value (typically `0`), which can remove padding or margin without any visible
error in the console.

Define a complete, unbroken spacing scale:

```css
:root {
  --space-1:  0.25rem;  /*  4px */
  --space-2:  0.5rem;   /*  8px */
  --space-3:  0.75rem;  /* 12px */
  --space-4:  1rem;     /* 16px */
  --space-5:  1.25rem;  /* 20px */
  --space-6:  1.5rem;   /* 24px */
  --space-7:  1.75rem;  /* 28px */
  --space-8:  2rem;     /* 32px */
  --space-10: 2.5rem;   /* 40px */
  --space-12: 3rem;     /* 48px */
  --space-16: 4rem;     /* 64px */
  --space-20: 5rem;     /* 80px */
  --space-24: 6rem;     /* 96px */
  --space-32: 8rem;     /* 128px */
}
```

### Fluid type scale

Use `clamp()` for all headline sizes. This prevents type from being simultaneously
too large on narrow phones and too small on tablets.

```css
--text-xl:  clamp(1.2rem,  2.8vw, 1.5rem);
--text-2xl: clamp(1.4rem,  3.5vw, 2rem);
--text-3xl: clamp(1.8rem,  4.5vw, 2.75rem);
--text-4xl: clamp(2.2rem,  6vw,   3.75rem);
```

At 390px viewport, `vw`-based preferred values are often smaller than the `min`
clamp argument — the minimum kicks in and the size stays fixed across the narrow
phone range. Verify at 375px that minimums do not produce sizes that feel too large
for the section's visual weight.

---

## Touch Target Standards

All interactive elements must meet WCAG 2.5.5 minimum 44×44px tap target size.

- Navigation hamburger buttons: `width: 44px; height: 44px`
- CTA buttons: sufficient `padding` to reach 44px height naturally
- Social/icon links in footers: wrap in a container sized to 44×44px minimum

---

## Horizontal Overflow Prevention

Always test for horizontal scroll at every breakpoint. Common causes:

- Fixed-width elements wider than the viewport (images, containers, absolute elements)
- `white-space: nowrap` on text without a width constraint
- Absolute-positioned decorative elements (large numerals, watermarks) extending
  beyond `overflow: hidden` containers
- Grid or flex children with `min-width` larger than available space

Quick diagnostic: add `* { outline: 1px solid red }` temporarily to identify
which element is causing overflow.

---

## CSS Grid Named-Area Mobile Fallback Risks

### The Production Failure Pattern

Named `grid-area` assignments on desktop layouts can cause catastrophic invisible-content failures
on Safari iOS when the corresponding `grid-template-areas` is absent at smaller breakpoints.
Desktop layouts appear correct. Mobile shows a solid background with no content.

This was encountered in production on a gallery page with a 3-column named-area grid. On real
iPhone Safari, the entire gallery section rendered as a blank dark rectangle. No images, no
captions — only the section background colour. CSS inspection in DevTools showed all `.gp-photo`
elements at `height: 0px`.

### Root Failure Mode

Named grid areas require a two-part CSS contract:

1. A container defines the named regions: `grid-template-areas: "featured portrait-a ..."`
2. Items claim those regions: `grid-area: featured`

When that contract breaks at a smaller breakpoint — the container drops to single-column and
`grid-template-areas` is removed, but items still declare `grid-area: featured` — the named
value becomes unresolved. The CSS specification says unresolved named areas fall back to
auto-placement. **Safari iOS does not reliably implement this fallback.** Items can all be placed
at row 1, column 1 — stacked on top of each other — rather than flowing down the column.

**Why this makes content invisible, not just mis-ordered:**

When all items stack at row 1, column 1, the explicit grid row auto-sizes from its intrinsic
content. If all children of the grid items are `position: absolute`, those children are removed
from normal flow and contribute zero intrinsic height to their grid container. The row height
collapses to 0px. The section background fills the viewport — appearing as a solid blank area
with no content of any kind.

**Why desktop masks the problem:**

Desktop defines `grid-template-areas` explicitly. All named areas resolve correctly. Items are
placed in distinct rows and columns with defined dimensions. The failure only occurs when
`grid-template-areas` is absent and named values become unresolved — which only happens at the
mobile breakpoint.

---

### Example Failure Pattern

```css
/* Desktop — multi-column editorial grid with named areas */
.gallery {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-auto-rows: 300px;
  grid-template-areas:
    "featured   portrait-a  portrait-b"
    "featured   landscape   landscape"
    "portrait-c landscape   landscape";
}

/* Items claim named areas */
.photo--featured   { grid-area: featured; }
.photo--portrait-a { grid-area: portrait-a; }

/* Image fill — absolutely positioned children */
.photo { position: relative; }
.photo__img {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
}
```

```css
/* Mobile breakpoint — single column, grid-template-areas removed */
@media (max-width: 899px) {
  .gallery {
    grid-template-columns: 1fr;
    /* grid-template-areas: intentionally absent for single column */
  }

  /* BUG: .photo--featured still has grid-area: featured
     Safari iOS may not resolve this to auto-placement.
     All items may stack at row 1, col 1.
     Children are position: absolute → contribute 0 intrinsic height.
     Row auto-sizes to 0px. Section appears blank black. */
}
```

### Intrinsic Height Collapse — The Chain of Events

The failure requires this specific combination:

1. **Named `grid-area` with no matching `grid-template-areas`** → Safari may mis-place all items at the same cell
2. **Grid container uses implicit or auto row sizing** → collapsed items auto-size the row from content
3. **All grid item children are `position: absolute`** → children are out of normal flow; contribute 0 intrinsic height
4. **Grid item has no explicit `height` or `min-height`** → height collapses to 0px
5. **Section background colour fills the space** → visually: solid background with no content

Any one of these conditions missing breaks the chain. If children are in normal flow, they
contribute intrinsic height and the collapse doesn't occur. If items have a `min-height`, the
floor overrides the collapse. The failure requires all five conditions simultaneously — which is
exactly the pattern used in image-fill grid layouts.

---

### Safe Mobile Fallback Pattern

When switching from a named-area desktop grid to a single-column mobile layout, always:

1. Explicitly reset `grid-area` to `auto` on all named items at the mobile breakpoint
2. Set a `min-height` floor independent of intrinsic content height
3. Apply both overrides at the same breakpoint that removes `grid-template-areas`

```css
/* ============================================================
   GRID NAMED-AREA MOBILE SAFETY RESET
   Required when desktop uses grid-template-areas and grid item
   children are position: absolute (image-fill pattern).
   ============================================================ */
@media (max-width: 899px) {
  .gallery {
    grid-template-columns: 1fr;
    /* grid-template-areas intentionally absent — single column */
  }

  /* Reset all named grid-area items to auto-placement.
     Safari iOS does not reliably fall back unresolved named areas.
     min-height prevents collapse when children are position: absolute. */
  .photo {
    grid-area: auto;
    min-height: 240px;
  }
}
```

`grid-area: auto` forces the browser to ignore the named assignment and use normal auto-placement.
`min-height: 240px` guarantees a height floor that does not depend on child intrinsic height —
safe regardless of whether children are in normal flow or absolutely positioned.

---

### When `!important` Is Required

If scroll-driven animation resets are applied in the same mobile block (common — both failure
modes can coexist on the same element), the `@supports (animation-timeline: scroll())` block can
override your mobile resets depending on cascade order. Use `!important` on animation-related
properties to guarantee the mobile block wins.

```css
@media (max-width: 899px) {
  .photo {
    grid-area: auto;
    min-height: 240px;
    animation: none !important;
    animation-timeline: auto !important;   /* shorthand may not reset sub-properties in all browsers */
    animation-range: normal !important;    /* explicit reset required */
    opacity: 1 !important;
    visibility: visible !important;
    transform: none !important;
  }
}
```

Note: `animation: none` shorthand does **not** reliably reset `animation-timeline` and
`animation-range` sub-properties in all browsers. Explicitly reset them both.

---

### QA Checklist — Grid Named-Area Layouts

Any page using `grid-template-areas` where grid item children are `position: absolute`
(the image-fill pattern) must pass this checklist before marking the page complete.

#### On-device or real-viewport runtime test

At 375px, 390px, 414px, and 430px:

- [ ] All grid items are visible — not collapsed to 0px height
- [ ] Items stack in predictable single-column flow, not overlapping
- [ ] Section does not show a solid background with no content
- [ ] Computed `height` on grid items is > 0px (inspect in DevTools)

#### CSS review

- [ ] All `grid-area: <named-value>` rules have a `grid-area: auto` override at the mobile breakpoint
- [ ] All grid items that use `position: absolute` children have a `min-height` floor at mobile

#### Principle

> A named `grid-area` value with no matching `grid-template-areas` is an unresolved CSS
> contract. Do not assume browsers implement the fallback consistently — Safari iOS is a
> confirmed production counterexample. Always explicitly reset `grid-area: auto` at the
> breakpoint where `grid-template-areas` is removed.

---

## Scroll-Driven Animation on Mobile

### The Production Failure Pattern

Scroll-driven reveal animations that begin at `opacity: 0` can render entire page sections
invisible on mobile, even when desktop layouts appear perfectly correct. This is not a
theoretical risk — it was encountered in production on a gallery page and is easy to miss
in a layout-only audit.

**Why desktop masks the problem:**

On desktop, the editorial grid is compact. Images are near or at the viewport on load,
the scroll distance to trigger the reveal is short, and the animation fires within the
first scroll interaction. Everything appears to work.

**Why mobile fails:**

Single-column stacking dramatically increases the total scroll height of a section. A
gallery that spans ~800px at desktop with a 3-column grid may span 2200px+ at mobile
with single-column 240px rows. Elements deep in the stack are far from the viewport
when the page loads. With `animation-fill-mode: both` and `from { opacity: 0 }`, every
element below the fold is invisible. The section background colour shows through —
appearing as a solid dark/black area with no content.

Layout audits alone cannot catch this. **Runtime viewport testing at each target width
is required.**

---

### The Specific CSS Risk

This failure mode is triggered by the combination of:

```css
@supports (animation-timeline: scroll()) {
  @keyframes reveal {
    from { opacity: 0; transform: translateY(10px); }
    to   { opacity: 1; transform: translateY(0); }
  }

  .section-item {
    animation: reveal linear both;
    animation-timeline: view();
    animation-range: entry 0% entry 28%;
  }
}
```

**Why elements stay invisible:**

`animation-fill-mode: both` applies the `from` keyframe state before the animation
range begins. `entry 0%` is the moment the element's top edge reaches the viewport's
bottom edge. Until that exact scroll position is reached, the element is at `opacity: 0`.

On a 240px tall mobile tile:
- `entry 28%` = 67px scrolled into view
- The element is invisible until 67px of it is visible

For elements 1000px below the fold, the user must scroll all the way to each individual
tile before it becomes visible. The section appears blank on first reach.

**The `@supports` guard does not protect against this.** It correctly prevents the
animation from applying on non-supporting browsers, but on Chrome 115+ and Safari 18+
(which do support `animation-timeline`) the guard passes and the failure occurs. The
guard only tells you whether the feature is available — not whether it is safe at all
viewport sizes.

---

### Mobile Standard: Disable Scroll-Driven Reveals Below the Editorial Threshold

Scroll-driven reveal animations are a desktop editorial effect. On mobile and tablet,
immediate content visibility must take priority.

**Default rule:** Disable scroll-driven opacity reveals at ≤899px (or whatever
breakpoint marks the transition from your multi-column editorial layout to
single-column or 2-column stacking).

```css
/* ============================================================
   SCROLL-DRIVEN ANIMATION MOBILE OVERRIDE
   Cancel opacity-based reveal on mobile/tablet where single-
   column stacking makes sections invisible before interaction.
   Desktop editorial grid retains the reveal effect.
   ============================================================ */
@media (max-width: 899px) {
  .section-item {
    animation: none;
    opacity: 1;
  }
}
```

**The threshold is the editorial layout breakpoint.** If your 3-column masonry
starts at ≥900px, disable the reveal at ≤899px. The reveal is an enhancement for
a compact editorial composition — it is not appropriate for a 2200px single-column
stack where the user cannot predict which scroll position makes content appear.

**Exceptions require deliberate design:**

Only retain scroll-driven reveals below 900px when:

1. The animation has been runtime-tested at 375px, 390px, 414px, and 430px
2. Content is visible before the user needs to scroll for it (no below-fold reveals)
3. The timing is short enough that nothing remains hidden for more than ~50px of scroll

---

### Reduced Motion: Reset the Container, Not Just the Children

The `prefers-reduced-motion` block must target **both** the animated container and its
children. The most common mistake is resetting child elements (images, captions) while
leaving the parent container's animation intact.

#### The bug

```css
/* WRONG — container .section-item still has opacity: 0 from scroll animation.
   Cancelling .section-item__img has no effect if the parent is invisible. */
@media (prefers-reduced-motion: reduce) {
  .section-item__img {
    animation: none !important;
    opacity: 1 !important;
  }
}
```

#### The correct pattern

```css
/* CORRECT — cancel the container animation AND restore its opacity.
   Then cancel child animations for completeness. */
@media (prefers-reduced-motion: reduce) {
  .section-item {
    animation: none !important;
    opacity: 1 !important;
    transform: none !important;
  }
  .section-item__img,
  .section-item__caption {
    animation: none !important;
    transition: none !important;
    opacity: 1 !important;
    transform: none !important;
  }
}
```

**Always trace the opacity chain from the root container down** when writing reduced
motion resets. If the container is `opacity: 0`, no child rule can make content visible.

---

### Mandatory QA Checklist — Animation Systems

Any page that uses scroll-driven animations, opacity reveals, or `animation-timeline`
must be tested with the following checklist **before marking the page complete.**

Layout review and CSS structure review are not substitutes for this checklist.

#### Runtime visibility check

At each of the following widths, scroll the full page from top to bottom:

| Width | Device reference |
|-------|-----------------|
| 375px | iPhone SE, oldest common viewport |
| 390px | iPhone 14 / 15 Pro |
| 414px | iPhone 11, XR, XS Max |
| 430px | iPhone 14 Plus / 15 Plus |

For each section that uses animated reveals, verify:

- [ ] Content is visible **before** the user begins scrolling to it
- [ ] No section shows a solid background colour with no content
- [ ] Scrolling to a section reveals content **within 50px of first scroll contact**
- [ ] No content remains permanently hidden (scroll to page bottom and confirm all sections rendered)

#### Reduced motion check

Enable `prefers-reduced-motion: reduce` in the browser or OS, reload, and verify:

- [ ] All content is immediately visible without any scrolling
- [ ] No containers are at `opacity: 0` (inspect computed styles if in doubt)
- [ ] No transforms shift content off-screen

#### Principle

> Layout audits are insufficient for animation systems. Runtime viewport testing
> is required. A section that appears correctly structured in CSS can be completely
> invisible to users at specific viewport sizes if scroll-driven opacity state is
> not explicitly reset.

---

## File Organisation Convention

Each page or section CSS file should have a clearly marked mobile override block
at the end, above the `prefers-reduced-motion` block:

```css
/* ============================================================
   MOBILE OVERRIDES — 375–430 px repair pass
   Desktop rules are untouched above this block.
   ============================================================ */

@media (max-width: 768px) { ... }
@media (max-width: 640px) { ... }
@media (max-width: 600px) { ... }
@media (max-width: 540px) { ... }
@media (max-width: 430px) { ... }


/* ============================================================
   REDUCED MOTION
   ============================================================ */

@media (prefers-reduced-motion: reduce) { ... }
```

Keeping overrides in a single labelled block (rather than scattered throughout the
file alongside their desktop rules) makes mobile QA and handoff significantly faster.