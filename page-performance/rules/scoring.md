# Scoring — page-performance `/100`

Every page-performance audit produces a `/100` score across 9 weighted categories. This rubric is the source of truth — when a phase finding is ambiguous, the evidence requirements below decide the deduction.

---

## Categories & weights

| # | Category | Weight | Maps to SOP phase |
|---|---|---:|---|
| 1 | Core Web Vitals (LCP · INP · CLS) — **field data preferred** | 25 | Phase 1, Phase 10 |
| 2 | Image optimisation (format · srcset · dims · LCP preload) | 15 | Phase 5 |
| 3 | Font + CSS + JS efficiency | 15 | Phase 6, Phase 7 |
| 4 | Resource prioritisation (preconnect · preload · defer · async) | 10 | Phase 4 |
| 5 | HTML payload size & validation | 10 | Phase 2 |
| 6 | Caching · compression · protocol (Brotli · HTTP/2-3 · Cache-Control) | 10 | Phase 3, Phase 9 |
| 7 | DOM size & semantic HTML | 8 | Phase 8 |
| 8 | Accessibility signals (alt · labels · ARIA · contrast) | 4 | Phase 10 |
| 9 | Structured-data ↔ performance (LCP image declared in schema) | 3 | Phase 10 |
| | **TOTAL** | **100** | |

**Bands**

| Band | Range | Meaning |
|---|---|---|
| Publish-ready | ≥ 85 | Ship as-is. Note top opportunities but no blocking fixes. |
| Targeted fixes | 70–84 | Action top-5 list before considering done. |
| Rework | < 70 | Significant performance debt — schedule a focused fix cycle, then re-audit. |

---

## How to score each category

Each row scores `0` to `weight`. Use the band table for that category, then add a free-form note citing the evidence (Lighthouse audit ID, header value, file size, CrUX bucket, DevTools panel output).

### 1. Core Web Vitals — 25 points

Field data (CrUX or GSC) **always overrides** lab data when both exist. Score the *worst-of-three* on the page's URL group.

| Field state (worst of LCP/INP/CLS) | Score |
|---|---:|
| All three "Good" in CrUX 28-day | 25 |
| Two "Good" + one "Needs improvement" | 20 |
| One "Good" + two "Needs improvement" | 15 |
| Any "Poor" (LCP > 4s · INP > 500ms · CLS > 0.25) | 8 |
| All "Poor" | 0 |

If no field data exists (low-traffic page), fall back to lab Lighthouse mobile:
- LCP ≤ 2.5s · INP/TBT ≤ 200ms · CLS ≤ 0.1 → 22 (lab ceiling, never max — lab is a diagnostic, not a ranking signal)
- One miss → 16
- Two misses → 10
- Three misses → 4

### 2. Image optimisation — 15 points

| Check | Points |
|---|---:|
| LCP image declared (any of: preload as image, `fetchpriority="high"`, in-DOM hero `<img>` with high-priority hint) | 4 |
| Hero image in next-gen format (AVIF or WebP) | 3 |
| All `<img>` tags have explicit `width` + `height` (or CSS `aspect-ratio`) | 3 |
| `srcset` + `sizes` on responsive images | 2 |
| Below-the-fold images use `loading="lazy"` | 2 |
| LCP image is **NOT** lazy-loaded (regression check) | 1 |

Deduct the full row if Lighthouse flags it red. Half if amber.

### 3. Font + CSS + JS efficiency — 15 points

| Check | Points |
|---|---:|
| Total CSS + JS ≤ 25 KB gzipped in ≤ 3 files (Encazip benchmark) | 4 |
| `font-display: swap` on every `@font-face` | 2 |
| Above-the-fold font file `preload`-ed | 2 |
| Variable font in use (or strong justification not to) | 2 |
| Coverage panel shows < 40 % unused CSS on any single file | 2 |
| No dev/JIT CDN script in production (e.g. `cdn.tailwindcss.com`) | 2 |
| Icon font subsetted to actually-used glyphs (when icon fonts present) | 1 |

### 4. Resource prioritisation — 10 points

| Check | Points |
|---|---:|
| `preconnect` to every third-party origin used above the fold | 2 |
| `preload as="image"` for the LCP image with `fetchpriority="high"` | 3 |
| `preload as="font" crossorigin` for the primary above-the-fold font | 2 |
| Non-critical `<script>` uses `defer` or `async` correctly | 2 |
| No overuse of `preload` (Lighthouse "Avoid chaining critical requests" passes) | 1 |

### 5. HTML payload size & validation — 10 points

| Check | Points |
|---|---:|
| HTML response body ≤ 450 KB (uncompressed) | 4 |
| W3C validator returns 0 errors | 2 |
| `<meta charset="UTF-8">` matches `Content-Type` header | 1 |
| Exactly one `<h1>`, one `<title>`, one canonical | 2 |
| Single `<main>` landmark | 1 |

### 6. Caching · compression · protocol — 10 points

| Check | Points |
|---|---:|
| `Content-Encoding: br` on text assets | 3 |
| HTTP/2 or HTTP/3 negotiated (DevTools Protocol column shows `h2` or `h3`) | 2 |
| Hashed static assets have `Cache-Control: max-age=31536000, immutable` | 2 |
| HTML uses `Cache-Control: no-cache` or short max-age with `ETag` revalidation | 2 |
| No mismatched compression between CDN and origin | 1 |

### 7. DOM size & semantic HTML — 8 points

| Check | Points |
|---|---:|
| DOM under 1,500 nodes | 3 |
| Correct landmark structure (`<header>` `<nav>` `<main>` `<footer>`) | 2 |
| Heading sequence descends without skipping levels | 2 |
| Visual section breaks align with semantic boundaries (VIPS) | 1 |

### 8. Accessibility signals — 4 points

| Check | Points |
|---|---:|
| Every meaningful `<img>` has descriptive `alt`; decorative use `alt=""` | 1 |
| Every form `<input>` linked to a `<label for="id">` | 1 |
| Icon-only buttons / links have `aria-label`; label content matches visible text | 1 |
| Lighthouse Accessibility score ≥ 95 + body text contrast ≥ 4.5:1 | 1 |

### 9. Structured data ↔ performance — 3 points

| Check | Points |
|---|---:|
| Page declares structured data (Article, Product, FAQ, Organization, etc.) | 1 |
| Declared `image` in structured data matches the rendered LCP element | 2 |

---

## Evidence requirements

Every scored row must include evidence in the "Notes / Evidence" column of the scorecard. Acceptable evidence types:

| Evidence type | Example |
|---|---|
| Lighthouse audit ID | `unused-css-rules: 1,420 ms wasted` |
| HTTP header | `content-encoding: br` (from `curl -sIL`) |
| File size | `tailwind.css = 23 KB gzipped` (Network panel) |
| CrUX bucket | `LCP 75th percentile = 2.1s — Good` (GSC CWV report) |
| DevTools panel reading | `Coverage panel: 68 % unused CSS on main.css` |
| Schema field | `Article.image = /hero.webp ✓ matches LCP element` |

"It looks fine" is not evidence. If a row cannot be evidenced, mark it `–` and explain why in the notes column — that may itself be a finding (e.g. "no CrUX data — page below 28-day traffic threshold").

---

## Top-5 prioritisation formula

Rank candidate fixes by `impact × ease`:

| Impact | Score | Ease | Score |
|---|---:|---|---:|
| Likely > 0.5s LCP drop OR > 0.1 CLS drop OR > 10pt score gain | 3 | < 1 hour dev work | 3 |
| Measurable but < 0.5s / < 0.1 / < 10pt | 2 | 1–4 hours | 2 |
| Marginal | 1 | > 4 hours or external dependency | 1 |

Sort by combined score descending; pick top 5. Tie-break by ease (cheaper first).

---

## When to update this rubric

- Google shifts a CWV threshold (e.g. INP target moves from 200ms). Update Category 1 table.
- A real audit surfaces a recurring pattern not captured here. Add a row to the relevant category, **don't invent a new category** — keep the 9-category shape stable so historical reports compare cleanly.
- Schema.org / structured-data guidance changes. Update Category 9.
