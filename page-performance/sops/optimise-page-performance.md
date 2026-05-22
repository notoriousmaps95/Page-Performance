# SOP — Optimise Page for Performance

**Source framework:** Manpreet Singh X Koray Tuğberk GÜBÜR - topical Map framework
**Version:** 1.0 — 2026-05-21
**Scope:** A single web page on a property you control — static HTML, CMS, or framework-rendered. Single-locale only.
**Output:** A `/100` audit report with category breakdown, top-5 prioritised actions, and a Next Actions table.

> **Guiding principle (Holistic SEO):** *"Every pixel, millisecond, byte, letter, and user matters for SEO."*
> Page speed is not a checkbox — Google treats it as a **health and trust signal** that compounds daily across every session, every crawl, every render. This SOP exists to turn that principle into a repeatable workflow.

---

## Section 1 — How to use this SOP

1. Run the **10 procedure phases** in order on the target page. Each phase has **Goal · Steps · Verification · Pitfalls**.
2. Score the page using the **Scorecard** in Section 4 — see [`../rules/scoring.md`](../rules/scoring.md) for the full /100 rubric and bands. Fill `Score` and `Notes` columns.
3. Derive the **Top 5 Prioritised Actions** (impact × ease) and the **Next Actions** table.
4. Save the completed report as `page-performance/reports/<page-slug>/audit-YYYY-MM-DD.md` so you can compare runs over time.
5. Numeric budgets per phase live in [`../rules/performance-budgets.md`](../rules/performance-budgets.md). Semantic + A11y rules (Phases 8 + 10) live in [`../rules/semantic-and-a11y.md`](../rules/semantic-and-a11y.md).

---

## Section 2 — Inputs & tools required

| Tool | Purpose |
|---|---|
| Target page URL (live, production) | Subject of the audit |
| Lighthouse (DevTools → Lighthouse, mobile + desktop) | Lab metrics, opportunities list |
| [PageSpeed Insights](https://pagespeed.web.dev/) | CrUX field data + Lighthouse in one view |
| [WebPageTest](https://www.webpagetest.org/) | Network-throttled real-device traces, filmstrip |
| Chrome DevTools — Network, Coverage, Performance, Rendering, Application panels | Bottleneck inspection |
| Google Search Console → Core Web Vitals report | 28-day field LCP/INP/CLS by URL group |
| [W3C HTML Validator](https://validator.w3.org/) | HTML correctness check |
| `curl -I <url>` and `curl --compressed -sIL <url>` | Inspect response headers, protocol, compression |
| Schema.org markup + [Rich Results Test](https://search.google.com/test/rich-results) | Verify structured data |

---

## Section 3 — Procedure (10 phases)

### Phase 1 — Baseline measurement

**Goal.** Capture a frozen "before" snapshot so every later change can be attributed.

**Steps.**
1. Run Lighthouse twice (mobile *and* desktop) with cache disabled. Record Performance score, LCP, INP, CLS, TBT, Speed Index, TTI.
2. Pull 28-day field data from CrUX / Search Console for the URL (or URL group). Field beats lab — Google ranks on field.
3. Run WebPageTest at "Slow 4G" mobile profile. Save the filmstrip + waterfall as artefacts.
4. Record current page weight (Network panel → bottom bar: requests, transferred, resources).
5. **Save a full-page screenshot.** In Chrome DevTools: `Cmd+Shift+P` → "Capture full size screenshot". Store it as `reports/<page-slug>/baseline-YYYY-MM-DD.png`. After every change during the audit cycle, capture the same shot again and visually diff. Lab metrics tell you it got faster; the screenshot tells you it didn't visibly break.
6. **Production Preview Protocol:** When auditing a local workspace, do **not** run the audit directly against a development server (e.g., Vite dev mode). Dev servers inject non-minified hot-reload scripts and WebSocket connections that trigger artificial score penalties (e.g., bf-cache failures, unminified JS warnings). Always compile the assets and launch a production preview server (e.g., `npm run preview`) to replicate production environments.

**Verification.** You have a numeric baseline for: LCP, INP, CLS, page weight (KB), request count, time-to-first-byte (TTFB), **plus a "before" full-page screenshot on disk**.

**Pitfalls.** Running Lighthouse on an active local dev server with HMR WebSockets instead of a static production preview; comparing lab to field directly (they're not the same scale); using cached runs; skipping the screenshot because "I'll remember what it looked like" — you won't.

---

### Phase 2 — HTML optimisation

**Goal.** Smallest possible HTML payload, valid markup, consistent encoding — so the crawler parses fast and renders predictably.

**Steps.**
1. Minify the HTML: strip comments, collapse whitespace, drop redundant attributes.
2. Validate at validator.w3.org. Fix every error; warnings are advisory but worth reviewing.
3. Enforce `UTF-8` in **both** the HTTP response header *and* `<meta charset="UTF-8">`. Mismatched encoding sends conflicting signals.
4. Target an HTML payload **under 450 KB** (Holistic SEO: Google's prior extraction limit). Above this, link discovery and rendering both degrade.
5. Confirm a single, correct `<title>`, one `<h1>`, and a unique canonical tag.

**Verification.**
```bash
curl -sIL https://example.com/page | grep -iE "content-type|content-encoding|content-length"
```
HTML file size visible in DevTools → Network → Doc filter. W3C validator returns 0 errors.

**Pitfalls.** Server sending `charset=ISO-8859-1` while the meta declares UTF-8; minifiers stripping conditional comments needed by legacy IE wrappers (rarely relevant now but check).

---

### Phase 3 — Compression & protocol

**Goal.** Transfer fewer bytes over fewer round-trips.

**Steps.**
1. Confirm **Brotli** (`br`) is enabled on the origin. Brotli outperforms Gzip by ~36 % on text assets (Holistic SEO).
2. Confirm **HTTP/2** or **HTTP/3** is negotiated. HTTP/2's binary framing + multiplexing removes head-of-line blocking that HTTP/1.1 suffered.
3. If using HTTP/2 Server Push, audit it — Holistic SEO warns that over-pushing *hurts* performance because the client may already have the resource cached.

**Verification.**
- DevTools → Network → right-click columns → enable **Protocol**. You should see `h2` or `h3`.
- Response headers should show `content-encoding: br`.

**Pitfalls.** CDN compresses but origin doesn't (or vice versa) — measure at the edge actually serving the user. Brotli not configured for dynamic responses, only static.

---

### Phase 4 — Resource prioritisation

**Goal.** Tell the browser, before it asks, what is critical.

**Steps.**
1. `<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>` for every third-party origin used above the fold. Handles DNS + TLS + TCP up-front. Replicate for any analytics or tag-manager origin actually used.
2. `<link rel="preload" as="image" href="/images/hero.avif" fetchpriority="high">` for the LCP image.
3. `<link rel="preload" as="font" type="font/woff2" href="/fonts/manrope-var.woff2" crossorigin>` for the primary above-the-fold font file.
4. Add `defer` to non-critical `<script>` tags so HTML parsing isn't blocked.
5. Use `async` only when the script has **no DOM dependency** and order doesn't matter (analytics, etc.). For UI logic (accordions, toggles) defer is safer.
6. Conditionally load polyfills via UA / feature detection — don't ship them to modern browsers.

**Verification.** Lighthouse "Preload Largest Contentful Paint image" and "Eliminate render-blocking resources" both green. LCP element shows in `Performance` panel within the first viewport paint.

**Pitfalls.** Preloading too much — anything preloaded competes with the LCP resource. Preload exactly the resources the browser would otherwise discover late.

---

### Phase 5 — Image optimisation

**Goal.** Visual fidelity at minimum bytes, with zero layout shift.

**Steps.**
1. **Format.** Convert to **AVIF** (best compression), fall back to **WebP**, then JPEG/PNG via `<picture>`.
2. **Responsive delivery.** Use `srcset` + `sizes` so phones don't download desktop-sized images.
3. **Dimensions.** Always set `width` and `height` attributes (or aspect-ratio in CSS). This eliminates CLS from images.
4. **Lazy-load below the fold.** `loading="lazy"` covers most cases. For *fine-grained* control (custom progressive loaders, scroll-triggered hydration), use the **Intersection Observer API** — Holistic SEO notes it has ~92 % browser support vs ~69 % for native `loading="lazy"`.
5. **Strip EXIF.** Remove camera metadata — adds KB, no SEO value.
6. **Placeholders.** Use LQIP (low-quality image placeholders) or `dominantColor` background so the user perceives progress instead of blank space.
7. **LCP image.** Add `fetchpriority="high"` and **preload it** (Phase 4 step 2). Do **not** lazy-load it.
8. **Programmatic Tag Injection.** For large multi-page static directories, write an automated Node.js script (using e.g., `image-size` or `jsdom`) to automatically scan all layout HTML files, fetch the real dimensions of local assets, and inject the correct `width` and `height` attributes to prevent CLS.
9. **Attribute Syntactic Cleanliness:** When programmatically modifying image attributes, ensure you strip any trailing slashes inside attribute declarations (e.g. converting `<img .../ width="...">` to `<img ... width="...">`). Trailing slashes within attributes are syntax errors that trigger browser parser warnings. Ensure slashes only appear (if at all) at the very end of the tag.

**Verification.** Lighthouse "Properly size images", "Serve images in next-gen formats", "Image elements have explicit width and height" all pass. CLS in field data < 0.1.

**Pitfalls.** Lazy-loading the LCP image (single most common LCP regression). Setting `decoding="async"` on the LCP image when it should render immediately.

---

### Phase 6 — Font optimisation

**Goal.** Text legible at first paint, smallest font payload.

**Steps.**
1. Audit current font files. Holistic SEO reduced **5 files (200+ KB) to 1 variable font (44 KB)** — aim for the same compression.
2. Where possible, use a **variable font** with weight/optical-size axes instead of 4–6 static cuts.
3. Apply `font-display: swap` to every `@font-face` rule. Prevents Flash of Invisible Text (FOIT) — the page renders system fallback while the web font loads.
4. **Preload only the font file actually used above the fold** (Phase 4 step 3). Don't preload italic weights the hero doesn't need.
5. Self-host critical fonts when possible — third-party DNS lookups cost time even with `preconnect`.
6. **Icon-font subsetting via `pyftsubset` or `woff2_decompress`.** When the page uses an icon font (Material Symbols, Lucide, etc.) request only the glyphs actually used:
   * Scan page code to extract active icon names. A simple single-line regex will miss multi-line icons or icons wrapped in whitespace. Use a robust Node.js parser script:
     ```javascript
     const regex = /material-symbols-outlined[^>]*>([\s\S]*?)<\/span>/gi;
     const icons = new Set();
     let match;
     while ((match = regex.exec(html)) !== null) {
       icons.add(match[1].trim());
     }
     ```
   * Extract unique icons, subset using standard tools (like `pyftsubset` with unicodes mapping to the selected icon names), and compress back to a highly-minimized local WOFF2 file.
   * **Mandatory pre-deploy assertion:** `count(icons_used_on_page) == count(icons_in_subset)` to prevent raw text fallback layout shifts.
7. **Relative Preloading Paths:** Always reference font assets (preloads and css url definitions) using relative paths (e.g. `fonts/...` or `../fonts/...`) instead of root-relative absolute paths (`/fonts/...`). Relative paths ensure fonts resolve correctly across multiple environments, including nested directory routing and local double-click file previews (`file://` scheme).

**Verification.** Lighthouse "Ensure text remains visible during webfont load" passes. DevTools → Coverage shows the font file fully used. **For icon-font subsetting, also visually inspect the page after the change — any missing icon will render as the literal icon name as text (e.g. `dentistry`, `local_hospital`, `check_circle`) and collapse the layout cell around it.**

**Pitfalls.**
- `font-display: block` (causes FOIT).
- Preloading a font that turns out to be an unused weight.
- Absolute paths in font references breaking under local preview folders.
- **Incomplete icon enumeration when subsetting an icon font** — a known regression pattern: an audit once shipped a subset covering 26 of 45 icons because the enumeration regex required the icon name to immediately follow the closing tag with no whitespace; 19 icons rendered as raw text on the live page. Always run the equality assertion above before deploying. Use the multi-line-safe `[\s\S]*?` regex shown in step 6.

---

### Phase 7 — CSS & JS efficiency

**Goal.** Smallest, most cache-friendly CSS/JS surface.

**Steps.**
1. **Segment CSS by page type** — Holistic SEO model:
   - File A — `header + footer` (shared across every page)
   - File B — `homepage-only` styles
   - File C — `subpage / template` styles

   This lets browsers cache shared rules across navigations.
2. **Target ≤ 25 KB total CSS/JS in ≤ 3 files** (Encazip benchmark: reduced 550+ KB / 8+ files → 22–25 KB / 3 files).
3. **Find unused CSS.** DevTools → Coverage → reload. Anything over 40 % unused is a candidate for removal or code-splitting.
4. **Cache headers.** Set `Cache-Control: max-age=31536000, immutable` on hashed static assets. Use `ETag` for revalidation on HTML.
5. **Minify** JS + CSS in production builds. Tree-shake unused exports.

**Common high-impact finding — Tailwind CDN.** When a site loads Tailwind via the CDN (`https://cdn.tailwindcss.com`), the CDN ships the **full JIT runtime (~3 MB)** to every visitor and is **not for production** per Tailwind's own docs. The fix is a compiled CSS file (`npx tailwindcss -i ./input.css -o ./dist/styles.css --minify`) — typical compiled payload is ~10–25 KB gzipped. Flag this in the report as a single-line top-5 action when detected.

**Verification.** Lighthouse "Reduce unused CSS", "Reduce unused JavaScript", "Enable text compression", "Serve static assets with an efficient cache policy" all green.

**Pitfalls.** Inlining critical CSS so aggressively that the HTML payload now exceeds 450 KB (Phase 2 budget). Cache-Control on HTML — don't make HTML immutable.

---

### Phase 8 — DOM size & semantic HTML

**Goal.** A small, semantically meaningful DOM that's cheap to lay out and easy to crawl.

**Steps.**
1. **Target under 1,500 DOM nodes** (Google recommendation, surfaced by Holistic SEO). Lighthouse warns above 1,500 and errors above 3,000.
2. Use **semantic tags**: `<header> <nav> <main> <article> <section> <aside> <footer> <figure>`. Visual structure and semantic structure should match.
3. **One `<h1>` per page**, one `<h2>` per top-level section. Heading order reflects the document outline.
4. **Visual section breaks should align with semantic boundaries** — Holistic SEO ties this to Google's Vision-based Page Segmentation (VIPS) algorithm: the renderer scores pages by visual blocks.
5. Remove deeply nested wrapper `<div>`s — they inflate node count and re-layout cost.

**Verification.** Lighthouse "Avoid an excessive DOM size" passes. Outline of the page (use the [HTML5 Outliner browser extension](https://github.com/h5o/h5o-chrome)) matches the visual hierarchy.

**Pitfalls.** Tailwind utility-heavy markup can balloon DOM depth — extract repeated structures into shared partials/templates where possible. Duplicated nav + footer markup across every page of a static site is a common DOM-budget killer.

---

### Phase 9 — Caching, PWA & service workers

**Goal.** Repeat visits should be near-instant.

**Steps.**
1. **`Cache-Control` + `ETag`** for all static assets. Long max-age for hashed files; short or revalidating for HTML.
2. **Service Worker** — register a worker that pre-caches the shell (HTML skeleton, CSS, font, logo). Returning visitors hit the worker, not the network.
3. **Web App Manifest** — `manifest.json` declaring name, icons, theme color, display mode. Enables "Add to Home Screen" on Android + iOS.
4. **Offline fallback page** — at minimum, a branded "you're offline" page so a flaky connection isn't a blank screen.

**Verification.** DevTools → Application → Service Workers shows active worker. Application → Manifest validates. Reloading with "Offline" checkbox in Network panel still serves the cached shell.

**Pitfalls.** Caching HTML too aggressively in the service worker — users see stale content after deploys. Use `stale-while-revalidate` or network-first for HTML.

---

### Phase 10 — Accessibility & structured-data signals tied to performance

**Goal.** Make sure performance gains don't break accessibility, and make Google's renderer see the same hierarchy the user does.

**Steps.**
1. **Image `alt` attributes.** Every meaningful image has descriptive `alt`. Decorative images use `alt=""`. Common regression: a templating step that emits `data-alt` instead of `alt` — verify on the rendered HTML, not the source.
2. **Form labels.** Every `<input>` paired with a `<label for="id">`. Placeholders are not labels.
3. **ARIA.** Add `role`, `aria-label`, `aria-labelledby`, `aria-describedby` where semantics aren't carried by native HTML.
4. **Colour contrast.** Body text ≥ 4.5:1, large text ≥ 3:1. Lighthouse accessibility audit catches most violations.
5. **LCP image in structured data.** Declare the LCP image inside `Article`, `Product`, `FAQPage`, or `Organization` schema so Google's renderer sees the same prominent image the user does. Holistic SEO ties this to unified authority signals.
6. **Skip-to-content link** for keyboard users — also helps screen readers reach `main` without re-reading the nav.

**Verification.** Lighthouse Accessibility score ≥ 95. Rich Results Test passes for declared schema. The image returned in `image` of the structured data is the same one rendered as LCP.

**Pitfalls.** Adding ARIA to elements that already have native semantics (`<button role="button">` is redundant and can confuse screen readers). Declaring a schema `image` that doesn't actually exist on the page.

---

## Section 4 — Scorecard (filled when running the SOP as an audit)

**Page audited:** `<URL>`
**Date:** `YYYY-MM-DD`
**Auditor:** `<name>`

| Category | Weight | Score (0–max) | Notes / Evidence |
|---|---|---|---|
| Core Web Vitals — LCP / INP / CLS (field data preferred) | 25 |  | Pull from CrUX / GSC. Pass = green on all three. |
| Image optimisation (format, srcset, dims, LCP preload) | 15 |  | Lighthouse opportunity values + Network panel scan. |
| Font + CSS + JS efficiency | 15 |  | Coverage panel, total transfer KB, file count. |
| Resource prioritisation (preconnect / preload / defer / async) | 10 |  | View-source check. |
| HTML payload size & validation | 10 |  | Doc size in Network panel. W3C error count. |
| Caching, compression, protocol (Brotli, HTTP/2-3, Cache-Control) | 10 |  | Response headers + Protocol column. |
| DOM size & semantic HTML | 8 |  | Lighthouse DOM count. Outline check. |
| Accessibility signals (alt, labels, ARIA, contrast) | 4 |  | Lighthouse a11y score. |
| Structured data tied to performance (LCP image in schema) | 3 |  | Rich Results Test + schema `image` check. |
| **TOTAL** | **100** |  |  |

### Top 5 Prioritised Actions

Rank by `(estimated impact on LCP/INP/CLS or score) × (ease of execution)`.

| # | Action | Phase | Impact | Effort | Owner | Deadline |
|---|---|---|---|---|---|---|
| 1 |  |  |  |  |  |  |
| 2 |  |  |  |  |  |  |
| 3 |  |  |  |  |  |  |
| 4 |  |  |  |  |  |  |
| 5 |  |  |  |  |  |  |

### Next Actions

| Action | Owner | Deadline | Verification |
|---|---|---|---|
|  |  |  |  |

---

## Section 5 — Appendix

### Source

This SOP is a workflow distillation of Koray Tuğberk GÜBÜR's Holistic SEO research study on Encazip.com: <https://www.holisticseo.digital/seo-research-study/holistic-seo>. The case study attributes a **155 % organic traffic increase over 6 months** and **110 % year-over-year growth** in part to the performance work catalogued above.

### Glossary

| Term | Meaning |
|---|---|
| **RAIL** | Response · Animation · Idle · Load — Google's user-perception performance model. 60 fps requires the browser to move pixels in ≤ 16 ms (≤ 10 ms working budget). |
| **LCP** | Largest Contentful Paint — when the largest above-the-fold element renders. Target ≤ 2.5 s. |
| **INP** | Interaction to Next Paint — replaces FID. How long the page takes to respond to a user input. Target ≤ 200 ms. |
| **CLS** | Cumulative Layout Shift — how much visible content unexpectedly moves. Target ≤ 0.1. |
| **TBT** | Total Blocking Time — sum of long-task blocking on the main thread. Lab proxy for INP. |
| **TTFB** | Time to First Byte — origin / CDN responsiveness. |
| **FOUT / FOIT** | Flash of Unstyled / Invisible Text — what happens during web-font load when `font-display` is misconfigured. |
| **Brotli** | Lossless compression algorithm by Google. ~36 % smaller than Gzip on text per Holistic SEO. |
| **VIPS** | Vision-based Page Segmentation — Google research algorithm that scores pages by visual blocks. Reinforces why semantic structure must match visual structure. |
| **LIFT** | Chris Goward's Layout Optimisation model — Locale, Information, Function, Trust. Referenced in Holistic SEO for aligning layout to search intent. |
| **LQIP** | Low-Quality Image Placeholder — tiny blurred version shown while the real image loads. |

### Worked findings — common patterns

A non-exhaustive set of patterns the audit surfaces repeatedly. Use as a starting checklist when an audit is short on time; do not treat as a substitute for running the phases.

| Phase | Common finding | Typical fix |
|---|---|---|
| 2 — HTML | No minification step in the deploy pipeline | Add a build-time HTML minifier (`html-minifier-terser` or equivalent). |
| 3 — Compression / protocol | CDN compresses but origin doesn't, or Brotli missing on dynamic responses | Configure Brotli at the edge actually serving the user; verify with `curl -sIL`. |
| 4 — Resource prioritisation | `preconnect` to font origins present, but no `preload` for the LCP image | Add `<link rel="preload" as="image" fetchpriority="high">` for the hero per template. |
| 5 — Images | `loading="lazy"` used; some pages emit `data-alt` instead of `alt` from a templating step | Convert heroes to AVIF + WebP; verify rendered HTML carries real `alt`. |
| 6 — Fonts | Two static cuts loaded from Google Fonts (e.g. one display + one body) | Self-host as a single variable font with the two axes you need. |
| 7 — CSS/JS | Tailwind via `cdn.tailwindcss.com` (~3 MB JIT runtime shipped to every visitor) | Compile to a static CSS file; remove the CDN `<script>`. Usually the single biggest win. |
| 8 — DOM | Utility-class markup inflates DOM; nav + footer duplicated across every page of a static site | Extract shared partials via a templating step; track DOM count per page. |
| 9 — Caching / PWA | Static host with no service worker | Add `manifest.json` + a minimal SW caching the shell with `stale-while-revalidate` for HTML. |
| 10 — A11y | `data-alt` instead of `alt`; `<input>` without `<label>`; `aria-label` doesn't include visible text | Fix at template level; re-verify with Lighthouse a11y audit. |

---

*End of SOP. To run as an audit, copy [`../templates/audit-report.md`](../templates/audit-report.md) to `page-performance/reports/<page-slug>/audit-YYYY-MM-DD.md`, walk Phases 1–10, then fill the scorecard.*
