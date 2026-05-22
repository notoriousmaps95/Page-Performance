# page-performance — Module

A 10-phase performance + semantic + accessibility audit for any single web page, derived from Koray Tuğberk GÜBÜR's Holistic SEO framework. Outputs a `/100` audit report with category breakdown, top-5 prioritised actions, and a Next Actions table.

**Source framework:** [Koray Tuğberk GÜBÜR — Holistic SEO Research Study (Encazip case)](https://www.holisticseo.digital/seo-research-study/holistic-seo)
**Version:** 1.0 — 2026-05-21
**Scope:** A single web page on a property the user controls — static HTML, CMS-rendered, or framework-rendered. Single-locale only.

---

## What this module does

Takes a live page URL, captures lab + field baselines, walks 10 audit phases (HTML payload → compression → resource prioritisation → images → fonts → CSS/JS → DOM/semantics → caching → a11y → CWV), and emits a dated Markdown report with:

- `/100` score across 9 weighted categories (see [rules/scoring.md](rules/scoring.md))
- Lab metrics (Lighthouse mobile + desktop) and field metrics (CrUX / GSC)
- Top-5 prioritised actions ranked by impact × ease
- Next Actions table with owner + deadline + verification

---

## How to invoke

```
Use the page-performance agent with url=https://<page-url>
```

Or directly:

```
Agent({
  subagent_type: "page-performance",
  prompt: "url=https://example.com/page"
})
```

The agent runs **Phase 1 (baseline measurement)** and **stops** with the captured numbers. Reply with confirmation (or "skip to audit") and it walks Phases 2–10 + emits the scorecard in a single continuation turn.

---

## Inputs

Required:
- **url** — full URL of the page being audited

Optional:
- **mode** — `quick` (lab-only, skip CrUX) | `full` (default — lab + field + WebPageTest)
- **competitors** — comma-separated URLs to benchmark against (max 3)

---

## Output layout

Written to `page-performance/reports/<page-slug>/`:

| File | Purpose |
|---|---|
| `audit-<YYYY-MM-DD>.md` | The /100 audit report (scorecard + top-5 + next actions) |
| `baseline-<YYYY-MM-DD>.png` | Full-page screenshot from Phase 1 |
| `lighthouse-mobile-<YYYY-MM-DD>.json` | Raw Lighthouse mobile report |
| `lighthouse-desktop-<YYYY-MM-DD>.json` | Raw Lighthouse desktop report |
| `notes.md` | Free-form findings, edge cases, follow-ups |

Page slug: derived from URL path. `https://example.com/services/hvac` → `example-com-services-hvac`.

---

## Scoring (per project standard — matches global CLAUDE.md "always /100")

**Weighted `/100` across 9 categories:**

| # | Category | Weight |
|---|---|---:|
| 1 | Core Web Vitals (LCP · INP · CLS, field-data preferred) | 25 |
| 2 | Image optimisation (format · srcset · dims · LCP preload) | 15 |
| 3 | Font + CSS + JS efficiency | 15 |
| 4 | Resource prioritisation (preconnect · preload · defer · async) | 10 |
| 5 | HTML payload size & validation | 10 |
| 6 | Caching · compression · protocol (Brotli · HTTP/2-3 · Cache-Control) | 10 |
| 7 | DOM size & semantic HTML | 8 |
| 8 | Accessibility signals (alt · labels · ARIA · contrast) | 4 |
| 9 | Structured-data ↔ performance link (LCP image in schema) | 3 |
| | **TOTAL** | **100** |

Bands: **≥ 85 publish-ready · 70–84 targeted fixes · < 70 rework**.

Full rubric: [rules/scoring.md](rules/scoring.md).

---

## Module layout

```
page-performance/
├── CLAUDE.md                              this file
├── .claude/agents/page-performance.md     (at repo root — subagent definition)
├── rules/
│   ├── workflow.md                        10-phase audit SOP — Phase order, dependencies
│   ├── semantic-and-a11y.md               canonical semantic HTML + A11y rule (merged)
│   ├── performance-budgets.md             numeric budgets — HTML <450KB, CSS/JS ≤25KB, DOM <1500
│   └── scoring.md                         /100 rubric, bands, evidence requirements
├── sops/
│   └── optimise-page-performance.md       the full 10-phase SOP with verification + pitfalls
├── templates/
│   └── audit-report.md                    scorecard skeleton — copied into reports/<slug>/
├── samples/                               reserved — case studies from real audits
└── reports/
    └── <page-slug>/
        └── audit-<YYYY-MM-DD>.md
```

---

## What this module covers (v1)

| Concern | Covered |
|---|---|
| Core Web Vitals diagnosis (LCP / INP / CLS) | ✓ |
| HTML / CSS / JS / font / image audit | ✓ |
| Resource prioritisation (preconnect/preload/defer/async) | ✓ |
| Brotli + HTTP/2/3 + caching headers verification | ✓ |
| DOM size + semantic landmark structure | ✓ |
| Accessibility — alt, labels, ARIA, contrast, keyboard | ✓ |
| Icon-font subsetting recipe | ✓ |
| Structured-data ↔ LCP linkage check | ✓ |
| Service worker + manifest review | ✓ |

---

## What this module does NOT cover

- **Full site audit** — this module audits one page at a time. For multi-page benchmarks, run it once per template (homepage, service, about, etc.).
- **Implementing the fixes** — the module produces the audit report and recommendations; the user (or a separate dev/agent) implements them.
- **CMS-specific deployment instructions** — recommendations are technology-agnostic.
- **A/B testing or conversion-rate analysis** — performance only.
- **Crawl-budget or log-file analysis** — that belongs to a future `crawl-audit` module.
- **Multi-locale performance comparison** — v1 single-locale only.

Keep each module narrow. One job, one answer.

---

## Maintenance

- **Add a new pitfall** to [sops/optimise-page-performance.md](sops/optimise-page-performance.md) when a real audit surfaces a regression pattern not yet documented (e.g. the icon-font subsetting enumeration bug already captured in Phase 6).
- **Add a budget** to [rules/performance-budgets.md](rules/performance-budgets.md) when a phase introduces a numeric target (KB, ms, count). Keep budgets in one place so they don't drift.
- **Re-check Google Web Vitals thresholds annually.** LCP/INP/CLS targets are stable today but Google has shifted them before (FID → INP in 2024).
- **Mirror the agent file** if you want to invoke from any project: copy `.claude/agents/page-performance.md` → `~/.claude/agents/page-performance.md`. Keep both in sync.
