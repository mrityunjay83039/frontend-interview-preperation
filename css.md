# 🎨 CSS & Modern Layouts Interview Guide

A comprehensive guide to modern CSS, layout architectures, the cascade, stacking contexts, animations, and responsive web design.

---

## 📑 Table of Contents
1. [The Box Model & Margin Collapsing](#1-the-box-model--margin-collapsing)
2. [CSS Specificity & The Cascade](#2-css-specificity--the-cascade)
3. [Positioning & Stacking Context (`z-index`)](#3-positioning--stacking-context-z-index)
4. [Flexbox Layout System](#4-flexbox-layout-system)
5. [CSS Grid Architecture](#5-css-grid-architecture)
6. [Responsive Design, Media Queries & Container Queries](#6-responsive-design-media-queries--container-queries)
7. [Modern Selectors (`:is()`, `:where()`, `:has()`)](#7-modern-selectors-is-where-has)
8. [Animations, Transitions & 60fps Rendering](#8-animations-transitions--60fps-rendering)
9. [CSS Architecture & Methodologies](#9-css-architecture--methodologies)

---

## 1. The Box Model & Margin Collapsing

### Box Sizing
- `box-sizing: content-box` (W3C default):
  $$\text{Rendered Width} = \text{width} + \text{padding-left} + \text{padding-right} + \text{border-left} + \text{border-right}$$
- `box-sizing: border-box`:
  $$\text{Rendered Width} = \text{width} \quad (\text{padding and border fit inside})$$

```css
*, *::before, *::after {
  box-sizing: border-box;
}
```

### Margin Collapsing
- Occurs when vertical margins of adjacent block elements touch.
- The browser collapses them to the single **largest** margin value (not their sum).
- **Does NOT collapse**:
  - Horizontal margins.
  - Elements inside Flexbox or CSS Grid containers.
  - Elements with `position: absolute` or `fixed`.
  - Elements separated by padding, borders, or clearings.

---

## 2. CSS Specificity & The Cascade

### Specificity Hierarchy
Specificity is calculated as a 4-part weight `(Inline, ID, Class/Attribute/Pseudo-class, Element/Pseudo-element)`:

| Selector Type | Example | Weight |
| :--- | :--- | :--- |
| **`!important`** | `color: red !important` | Overrides all standard specificity |
| **Inline Styles** | `<div style="color: blue">` | `1, 0, 0, 0` |
| **ID Selectors** | `#header`, `#nav` | `0, 1, 0, 0` |
| **Classes, Attributes, Pseudo-classes** | `.btn`, `[type="text"]`, `:hover`, `:nth-child(2)` | `0, 0, 1, 0` |
| **Elements, Pseudo-elements** | `div`, `p`, `::before`, `::after` | `0, 0, 0, 1` |
| **Universal & Combinators** | `*`, `>`, `+`, `~` | `0, 0, 0, 0` |

---

## 3. Positioning & Stacking Context (`z-index`)

### `position` Values
- `static`: Default flow. Top/right/bottom/left and `z-index` have no effect.
- `relative`: Positioned relative to its normal in-flow position without removing space.
- `absolute`: Removed from document flow; positioned relative to its nearest **positioned ancestor** (ancestor with `position != static`).
- `fixed`: Removed from flow; positioned relative to viewport (or containing block created by `transform`/`filter` on an ancestor).
- `sticky`: Hybrid of `relative` and `fixed`. Sticks when reaching a scroll threshold within its parent container.

### Stacking Context Triggers
A new stacking context is formed by:
1. Root `<html>` element.
2. Element with `position: absolute` or `relative` AND `z-index` other than `auto`.
3. Element with `position: fixed` or `sticky`.
4. Element with `opacity < 1`.
5. Element with `transform`, `filter`, `perspective`, `clip-path` other than `none`.
6. Element with `isolation: isolate`.

> 💡 **Golden Rule**: Elements in a lower stacking context can **never** appear above elements in a higher stacking context, no matter how large their `z-index` value is.

---

## 4. Flexbox Layout System

One-dimensional layout (row OR column).

### Container Properties
- `display: flex | inline-flex`
- `flex-direction: row | row-reverse | column | column-reverse`
- `justify-content`: Aligns along the **main axis** (`flex-start`, `center`, `space-between`, `space-around`, `space-evenly`).
- `align-items`: Aligns along the **cross axis** (`stretch`, `center`, `flex-start`, `flex-end`, `baseline`).
- `flex-wrap: nowrap | wrap | wrap-reverse`
- `gap: 16px`

### Item Properties
- `flex-grow`: Proportion of available remaining space to absorb (default: `0`).
- `flex-shrink`: Ability to shrink when space is insufficient (default: `1`).
- `flex-basis`: Initial default size before distributing remaining space (default: `auto`).
- Shorthand: `flex: 1 1 0%` (grow, shrink, basis).
- `align-self`: Overrides container's `align-items` for an individual item.

---

## 5. CSS Grid Architecture

Two-dimensional layout (rows AND columns simultaneously).

```css
.grid-container {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 24px;
}
```

### Auto-fill vs Auto-fit
- `auto-fill`: Fills the row with as many column tracks as can fit, even if empty.
- `auto-fit`: Fits existing tracks into the container, collapsing empty tracks and expanding populated tracks to fill remaining space.

### Holy Grail Layout with Grid Template Areas
```css
.app-layout {
  display: grid;
  grid-template-areas:
    "header header header"
    "nav    main   aside"
    "footer footer footer";
  grid-template-rows: 60px 1fr 60px;
  grid-template-columns: 200px 1fr 200px;
  min-height: 100vh;
}

header { grid-area: header; }
nav    { grid-area: nav; }
main   { grid-area: main; }
aside  { grid-area: aside; }
footer { grid-area: footer; }
```

---

## 6. Responsive Design & Modern Media/Container Queries

### Fluid Typography & Spacing
```css
/* clamp(minimum, ideal/scalable, maximum) */
h1 {
  font-size: clamp(1.75rem, 4vw + 1rem, 3.5rem);
}
```

### Container Queries (`@container`)
Styles elements based on their **parent container's width**, not the global viewport:
```css
.card-wrapper {
  container-type: inline-size;
  container-name: card;
}

@container card (min-width: 400px) {
  .card {
    display: flex;
    flex-direction: row;
  }
}
```

---

## 7. Modern Selectors (`:is()`, `:where()`, `:has()`)

### `:is()` vs `:where()`
- `:is(h1, h2, h3)`: Specificity equals the **most specific** selector in the list.
- `:where(h1, h2, h3)`: Specificity is **always zero** (`0,0,0,0`), making it ideal for base resets.

### `:has()` (The Parent Selector)
Selects parent elements based on their children:
```css
/* Style article card only when it contains a featured image */
article:has(img.featured) {
  grid-column: span 2;
}

/* Style form label when input inside is invalid */
label:has(+ input:invalid) {
  color: red;
}
```

---

## 8. Animations, Transitions & 60fps Rendering

- [ ] Stick to **GPU-composited** properties: `transform` and `opacity`.
- [ ] Avoid animating geometry properties (`width`, `height`, `left`, `margin`) because they cause layout reflow every frame.
- [ ] Use `will-change: transform` conservatively to hint to the browser to promote the element to its own GPU layer.
- [ ] Support user accessibility preferences:
  ```css
  @media (prefers-reduced-motion: reduce) {
    *, *::before, *::after {
      animation-duration: 0.01ms !important;
      transition-duration: 0.01ms !important;
    }
  }
  ```

---

## 9. CSS Architecture & Methodologies

- [ ] **BEM (Block Element Modifier)**: `.card`, `.card__title`, `.card--highlighted`.
- [ ] **CSS Modules**: Locally scoped class names generated at build time (`.header_xyz123`).
- [ ] **CSS Variables (Custom Properties)**:
  - Cascading, dynamic at runtime via JS (`document.documentElement.style.setProperty('--primary', '#ff0000')`).
  - Supports theming (Dark / Light mode).
- [ ] **Utility-First (Tailwind CSS)**: High velocity, zero runtime overhead, eliminates dead CSS.
