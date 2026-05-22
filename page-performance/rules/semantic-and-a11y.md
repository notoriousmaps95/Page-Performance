# Semantic HTML & Accessibility Rules

The canonical semantic + A11y reference for the page-performance audit. Phases 8 and 10 of the audit SOP measure compliance against this file. Anything not stated here is out of scope for the audit — keep this file narrow and authoritative.

**Why this matters for SEO, not just A11y.** Google's renderer parses pages by semantic landmarks, visual blocks (VIPS algorithm), and image dimensions. Semantically clean pages render faster (browsers calculate layout faster on flat semantic trees than on deeply nested `<div>` trees), suffer fewer layout shifts, and signal content priority to the crawler in one pass.

---

## 1. Landmark structure

Every audited page MUST contain these top-level HTML5 landmarks:

| Element | Purpose | Quantity |
|---|---|---|
| `<header>` | Branding + nav wrapper | 1 |
| `<nav>` | Navigation links (inside or below `<header>`) | ≥ 1 |
| `<main>` | Primary page content | **Exactly 1** |
| `<footer>` | Copyright, site-wide links, legal | 1 |

**Anti-pattern.** Hero, breadcrumb, or contact sections placed directly under `<body>` without a `<main>` wrapper. The renderer cannot tell which block is the primary content area; the audit scores this as a Phase 8 violation.

**Correct shape:**

```html
<body>
  <header>
    <nav><!-- links --></nav>
  </header>
  <main>
    <!-- all primary content -->
  </main>
  <footer><!-- legal, copyright --></footer>
</body>
```

---

## 2. Heading sequence (document outline)

- **Exactly one `<h1>` per page** — typically the hero title.
- Headings descend sequentially: `<h1>` → `<h2>` → `<h3>` → `<h4>`. **Never skip a level** (jumping `<h2>` → `<h4>` breaks the outline).
- Footer / sidebar section titles continue the page's hierarchy. If body content tops out at `<h2>`, footer section headings are `<h3>` — not `<h4>` or `<h5>`.
- **Heading tags carry semantic weight, not visual weight.** Don't pick `<h3>` because you wanted "medium-large text". Use a CSS class for the size; pick the heading tag for its place in the outline.

Verify with the [HTML5 Outliner](https://github.com/h5o/h5o-chrome) browser extension — the outline should match the visual hierarchy.

---

## 3. Interactive element accessibility (A11y names + label matching)

Any clickable, tappable, or focusable element must carry an accessible text name.

| Pattern | Rule |
|---|---|
| `<button>` with visible text | No extra ARIA needed — visible text *is* the accessible name. |
| `<button>` with icon only | Add `aria-label`. Icon SVG should have `aria-hidden="true"`. |
| `<a>` with icon only (WhatsApp, social) | Add `aria-label` describing destination. |
| `<button>` or `<a>` showing a short label (e.g. a number, a single letter) | `aria-label` MUST include the visible text. A button showing "1" with `aria-label="Step One"` triggers Lighthouse's "label-content-name-mismatch" rule. Correct: `aria-label="1 - Step One"`. |
| `<button>` for forms | Set explicit `type="submit"` or `type="button"` — the default `type="submit"` will submit any parent form unexpectedly. |

**Examples:**

```html
<!-- icon-only link -->
<a href="https://wa.me/..." aria-label="Chat on WhatsApp">
  <svg aria-hidden="true">...</svg>
</a>

<!-- numbered hotspot — label includes the visible number -->
<button type="button" aria-label="1 - Introduction">1</button>

<!-- explicit type prevents accidental form submit -->
<button type="button">Open dialog</button>
```

---

## 4. Form labels

| Element | Rule |
|---|---|
| `<input>` | Linked to a `<label for="id">` via matching `id` and `for`. |
| `<textarea>` | Same — `<label for="id">`. |
| `<select>` | Same. |
| Placeholder text | **Never a substitute for a label.** Placeholders disappear on focus; users with cognitive load lose the field's purpose. |
| Required-field indicator | Use `required` attribute. Visible asterisk lives in the `<label>`, not the placeholder. |
| Optional-field indicator | If used, must be solid high-contrast (≥ 4.5:1), not `opacity-60` light grey. |

```html
<label for="user-email">Email address</label>
<input id="user-email" type="email" placeholder="you@example.com" required>
```

---

## 5. Layout shift prevention (CLS)

| Asset | Rule |
|---|---|
| `<img>` | Always set `width` + `height` attributes (the **native** aspect ratio, not the display size). Browser reserves the box before download. |
| `<iframe>` | Same — explicit `width` + `height` or a CSS `aspect-ratio` wrapper. |
| `<video>` | Set `width` + `height` + `poster`. |
| Web-font swap | Use `font-display: swap` and metric-matched fallback (`size-adjust`, `ascent-override`, `descent-override`) to minimise the layout shift when the web font loads. |
| Late-injected content (ads, modals, banners) | Reserve space via `min-height` or `aspect-ratio` so the body doesn't reflow when they appear. |
| Animations | Animate only `transform` and `opacity`. Never animate `top`, `left`, `width`, `height`, `margin` — those force layout recalculation and trigger CLS. |

**Below-the-fold images:** `loading="lazy" decoding="async"`.

**LCP image:** preload + `fetchpriority="high"` + `decoding="async"`. **Never** `loading="lazy"` on the LCP image — single most common LCP regression.

**Programmatic dimension injection.** For multi-page static directories, write a Node script (`image-size` or `jsdom`) that scans layout HTML, reads native image dimensions, and injects `width` + `height`. Strip any trailing slashes inside the tag — `<img ... / width="...">` is a parser error.

---

## 6. Colour contrast

| Text type | Minimum ratio |
|---|---:|
| Body text (< 18.66px / 14pt regular or 24px / 18pt bold) | **4.5 : 1** |
| Large text (≥ 18.66px regular or 24px bold) | **3 : 1** |
| Icon + graphical UI element edges | **3 : 1** |

**Common failures to look for:**

- Light-teal text on a teal background (badges, tags).
- White-on-amber CTAs with low-opacity hover states.
- `opacity-60` greys on white backgrounds for "(optional)" subscripts.
- Placeholder text below the 4.5:1 threshold against the input background.

Prefer **solid colour shifts** over opacity tweaks — opacity compounds against background and is harder to keep above 4.5:1 across light/dark themes.

---

## 7. ARIA hygiene

- **Use native semantics first.** `<button>` already exposes `role="button"` — adding it manually is redundant and can confuse screen readers.
- **`aria-label` vs `aria-labelledby`.** Use `aria-labelledby` when the label is visible elsewhere on the page (point at its `id`). Use `aria-label` only when no visible label exists.
- **Live regions.** Status messages (toast, error banner) use `role="status"` (polite) or `role="alert"` (assertive). Don't use `aria-live` without one of these roles.
- **Skip-to-content link.** First focusable element in the page jumps to `<main>`. Visible on focus, hidden otherwise. Helps keyboard users *and* screen readers skip the nav.

```html
<a href="#main" class="sr-only focus:not-sr-only">Skip to main content</a>
...
<main id="main"><!-- ... --></main>
```

---

## 8. Structured data ↔ semantic structure

This is the bridge between semantic HTML and the structured-data the page declares. Verified in Phase 10 of the audit SOP.

| Schema field | Must match |
|---|---|
| `Article.headline`, `Product.name`, `FAQPage.name` | The `<h1>` content (or close to it — exact match preferred) |
| `Article.image`, `Product.image`, `Organization.logo` | The rendered LCP element (or be one of the page's actual rendered images) |
| `BreadcrumbList.itemListElement` | The visible breadcrumb trail's anchor text + URLs |
| `FAQPage.mainEntity[].name` | Visible question text on the page (exact match) |

**Anti-pattern.** Declaring an `Article.image` URL that doesn't exist on the page, or pointing it at a 1×1 tracking pixel. Google's renderer compares declared schema to rendered DOM — mismatches degrade structured-data trust over time.

---

## Verification commands (used by Phase 8 + Phase 10)

```bash
# Single <h1> per page
curl -s <url> | grep -ciE "<h1[ >]"
# Should return: 1

# UTF-8 encoding consistency
curl -sI <url> | grep -i "content-type"
# Should include: charset=utf-8

# Heading sequence — manually verify with HTML5 Outliner browser extension
# https://github.com/h5o/h5o-chrome

# Lighthouse a11y score — should be ≥ 95
# Chrome DevTools → Lighthouse → Accessibility only → Mobile
```

---

## Cross-references

| Need | File |
|---|---|
| Full Phase 8 (DOM size + semantic) and Phase 10 (a11y) steps | [`sops/optimise-page-performance.md`](../sops/optimise-page-performance.md) |
| How these rules score against the /100 rubric | [`rules/scoring.md`](scoring.md) — Categories 7, 8, 9 |
| Numeric DOM + contrast budgets | [`rules/performance-budgets.md`](performance-budgets.md) |
