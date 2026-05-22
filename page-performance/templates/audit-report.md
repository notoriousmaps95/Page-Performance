# Page Performance Audit — `<PAGE_URL>`

**Date:** `YYYY-MM-DD`
**Auditor:** `<name>`
**Mode:** `quick` | `full`
**SOP version:** 1.0 (2026-05-21)

---

## TL;DR

`<2–3 sentences. Total score, biggest leverage point, biggest risk if not fixed.>`

**Score:** `<XX> / 100` — **Band:** `Publish-ready (≥85)` | `Targeted fixes (70–84)` | `Rework (<70)`

---

## Baseline measurements (Phase 1)

### Lab — Lighthouse mobile

| Metric | Value | Target | Status |
|---|---:|---:|:---:|
| Performance score | | ≥ 90 | |
| LCP | | ≤ 2.5 s | |
| INP / TBT | | ≤ 200 ms | |
| CLS | | ≤ 0.1 | |
| Speed Index | | ≤ 3.4 s | |
| TTI | | ≤ 3.8 s | |
| FCP | | ≤ 1.8 s | |

### Lab — Lighthouse desktop

| Metric | Value |
|---|---:|
| Performance score | |
| LCP | |
| INP / TBT | |
| CLS | |

### Field — CrUX / GSC (28-day)

| Metric | 75th percentile | Bucket |
|---|---:|---|
| LCP | | Good / Needs improvement / Poor |
| INP | | Good / Needs improvement / Poor |
| CLS | | Good / Needs improvement / Poor |

*If no field data: page below 28-day traffic threshold for CrUX inclusion.*

### Page weight

| Metric | Value | Budget |
|---|---:|---:|
| HTML (uncompressed) | KB | ≤ 450 KB |
| CSS (gzipped, total) | KB | ≤ 25 KB |
| JS (gzipped, total) | KB | ≤ 25 KB |
| Image weight (transferred) | KB | — |
| Font weight (transferred) | KB | ≤ 50 KB per file |
| Total transferred | KB | ≤ 1,500 KB |
| Request count | | ≤ 50 |
| DOM nodes | | < 1,500 |
| TTFB | ms | ≤ 600 ms |

Baseline screenshot: `baseline-YYYY-MM-DD.png`

---

## Findings by phase

### Phase 2 — HTML optimisation
`<finding · evidence · suggested fix>`

### Phase 3 — Compression & protocol
`<finding · evidence · suggested fix>`

### Phase 4 — Resource prioritisation
`<finding · evidence · suggested fix>`

### Phase 5 — Image optimisation
`<finding · evidence · suggested fix>`

### Phase 6 — Font optimisation
`<finding · evidence · suggested fix>`

### Phase 7 — CSS & JS efficiency
`<finding · evidence · suggested fix>`

### Phase 8 — DOM size & semantic HTML
`<finding · evidence · suggested fix>`

### Phase 9 — Caching / PWA / service worker
`<finding · evidence · suggested fix>`

### Phase 10 — Accessibility + structured-data ↔ performance
`<finding · evidence · suggested fix>`

---

## Scorecard

| # | Category | Weight | Score | Notes / Evidence |
|---|---|---:|---:|---|
| 1 | Core Web Vitals (LCP · INP · CLS, field preferred) | 25 |  |  |
| 2 | Image optimisation (format · srcset · dims · LCP preload) | 15 |  |  |
| 3 | Font + CSS + JS efficiency | 15 |  |  |
| 4 | Resource prioritisation (preconnect · preload · defer · async) | 10 |  |  |
| 5 | HTML payload size & validation | 10 |  |  |
| 6 | Caching · compression · protocol (Brotli · HTTP/2-3 · Cache-Control) | 10 |  |  |
| 7 | DOM size & semantic HTML | 8 |  |  |
| 8 | Accessibility signals (alt · labels · ARIA · contrast) | 4 |  |  |
| 9 | Structured-data ↔ performance (LCP image in schema) | 3 |  |  |
| | **TOTAL** | **100** |  |  |

---

## Top 5 Prioritised Actions

Ranked by `impact × ease` (see scoring rubric for the formula).

| # | Action | Phase | Impact | Effort | Owner | Deadline |
|---|---|---|---|---|---|---|
| 1 |  |  |  |  |  |  |
| 2 |  |  |  |  |  |  |
| 3 |  |  |  |  |  |  |
| 4 |  |  |  |  |  |  |
| 5 |  |  |  |  |  |  |

---

## Next Actions

| Action | Owner | Deadline | Verification |
|---|---|---|---|
|  |  |  |  |

---

## Artefacts

- `lighthouse-mobile-YYYY-MM-DD.json` — full Lighthouse mobile report
- `lighthouse-desktop-YYYY-MM-DD.json` — full Lighthouse desktop report
- `baseline-YYYY-MM-DD.png` — full-page screenshot
- `notes.md` — free-form findings, edge cases, follow-ups

---

## References

- Workflow: [`../../rules/workflow.md`](../../rules/workflow.md)
- Rubric: [`../../rules/scoring.md`](../../rules/scoring.md)
- Budgets: [`../../rules/performance-budgets.md`](../../rules/performance-budgets.md)
- Semantic + A11y rules: [`../../rules/semantic-and-a11y.md`](../../rules/semantic-and-a11y.md)
- Full SOP: [`../../sops/optimise-page-performance.md`](../../sops/optimise-page-performance.md)
