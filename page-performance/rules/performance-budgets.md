# Performance Budgets

All numeric budgets for the page-performance audit live here. When a phase says "should be under X", `X` is defined in this file. Single source of truth so budgets don't drift across the SOP, scoring, and report.

---

## Core Web Vitals (field data — CrUX / GSC)

These are Google's published "Good" thresholds at the 75th percentile of the URL group. **Field beats lab** — these targets are what the audit ultimately measures against.

| Metric | "Good" | "Needs improvement" | "Poor" |
|---|---:|---:|---:|
| **LCP** — Largest Contentful Paint | ≤ 2.5 s | ≤ 4.0 s | > 4.0 s |
| **INP** — Interaction to Next Paint | ≤ 200 ms | ≤ 500 ms | > 500 ms |
| **CLS** — Cumulative Layout Shift | ≤ 0.1 | ≤ 0.25 | > 0.25 |

---

## Lab proxies (Lighthouse mobile)

Used when no field data is available, or as a leading indicator during fixes.

| Metric | Target |
|---|---:|
| Performance score (Lighthouse mobile) | ≥ 90 |
| FCP — First Contentful Paint | ≤ 1.8 s |
| LCP (lab) | ≤ 2.5 s |
| TBT — Total Blocking Time | ≤ 200 ms |
| CLS (lab) | ≤ 0.1 |
| Speed Index | ≤ 3.4 s |
| TTI — Time to Interactive | ≤ 3.8 s |

---

## Payload budgets

| Asset class | Budget | Source |
|---|---:|---|
| HTML response body (uncompressed) | ≤ 450 KB | Holistic SEO — Google's prior extraction limit; above this, link discovery + rendering degrade |
| Total CSS + JS (gzipped) | ≤ 25 KB | Encazip case — reduced 550+ KB across 8+ files to 22–25 KB across 3 files |
| Total CSS + JS file count | ≤ 3 files | Same — fewer files = better cache hit ratio |
| Single icon font (subsetted) | ≤ 5 KB | Encazip case — full Material Symbols to subsetted woff2 |
| Single web font file | ≤ 50 KB (variable) / ≤ 25 KB (per static weight) | Variable fonts collapse 5+ static cuts into 1 file |
| Hero image (LCP) — JPEG/PNG | Avoid | Use AVIF or WebP |
| Hero image (LCP) — AVIF | ≤ 100 KB | Practical ceiling for an above-the-fold image |
| Hero image (LCP) — WebP | ≤ 150 KB | Practical ceiling — WebP heavier than AVIF |
| Below-fold images | Lazy-loaded, no per-image budget | Sum to total page weight only |
| Total page weight (transferred) | ≤ 1.5 MB | Practical mobile budget at the 75th-percentile connection |
| Total request count | ≤ 50 | Beyond this, HTTP/2 multiplexing benefits flatten |

---

## DOM budgets

| Metric | Target | Source |
|---|---:|---|
| Total DOM nodes | < 1,500 | Lighthouse warns above 1,500, errors above 3,000 |
| Maximum DOM depth | < 32 | Lighthouse "Avoid an excessive DOM size" guidance |
| Maximum child elements per parent | < 60 | Same |

---

## Transport + protocol budgets

| Check | Target |
|---|---|
| `Content-Encoding` | `br` (Brotli) on text assets; `gzip` acceptable but loses ~36% vs Brotli |
| HTTP protocol | `h2` or `h3` (DevTools Network → Protocol column) |
| TTFB | ≤ 600 ms (origin); ≤ 200 ms (CDN-cached) |
| TLS handshake | TLS 1.3 |

---

## Caching budgets

| Asset class | `Cache-Control` |
|---|---|
| Hashed static assets (`main.a1b2c3.css`) | `max-age=31536000, immutable` |
| Unhashed static assets | `max-age=86400` + `ETag` |
| HTML | `no-cache` OR `max-age=0, must-revalidate` + `ETag` |
| API JSON | Case-by-case — typically `no-cache` |

Service worker (if present):
- HTML — network-first OR stale-while-revalidate (never cache-first; deploy-stale risk)
- CSS/JS/fonts — cache-first
- Images — stale-while-revalidate

---

## Accessibility budgets

| Check | Target |
|---|---|
| Lighthouse Accessibility score | ≥ 95 |
| Body text contrast ratio | ≥ 4.5 : 1 |
| Large text contrast ratio | ≥ 3 : 1 |
| Form fields linked to labels | 100% |
| Icon-only interactive elements with `aria-label` | 100% |
| `<h1>` per page | Exactly 1 |
| `<main>` per page | Exactly 1 |
| Heading-level skips | 0 |

---

## When to override a budget

These are working targets, not laws. A justified override is fine when:

| Budget | Override justification |
|---|---|
| HTML ≤ 450 KB | Page is intentionally a single-HTML offline-first artefact (e.g. a self-contained doc). |
| CSS+JS ≤ 25 KB | The site uses a heavy framework that earns its weight via measurable conversion lift, and the audit confirms it isn't the LCP blocker. |
| DOM < 1,500 | The page is a data table or product listing where DOM count tracks content volume. Then track depth + child-per-parent instead. |
| LCP ≤ 2.5s | Never. This is Google's threshold. If the page can't hit it, the audit's job is to explain why. |

Always note the override + justification in the scorecard's evidence column. The audit report should never hide a missed budget.
