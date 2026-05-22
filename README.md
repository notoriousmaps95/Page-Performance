# Page-Performance

A Claude Code subagent that runs a 10-phase performance + semantic + accessibility audit on any single web page, derived from Koray Tuğberk GÜBÜR's [Holistic SEO Research Study](https://www.holisticseo.digital/seo-research-study/holistic-seo).

Outputs a `/100` audit report with category breakdown, top-5 prioritised actions, and a Next Actions table — ready to hand to a developer.

**Version:** 1.0
**Scope:** Single page on a property you control. Static HTML, CMS-rendered, or framework-rendered. Single-locale only.
**Requires:** [Claude Code](https://claude.com/claude-code) CLI or the VS Code extension.

---

## What you get

Run the agent against a URL and you get a dated Markdown report containing:

- **`/100` score** across 9 weighted categories (Core Web Vitals · images · fonts/CSS/JS · prioritisation · HTML payload · caching/protocol · DOM/semantics · accessibility · structured-data link).
- **Lab + field measurements** — Lighthouse mobile + desktop, CrUX / GSC 28-day field pull, WebPageTest filmstrip.
- **Top-5 prioritised actions** ranked by impact × ease.
- **Next Actions table** with owner + deadline + verification.
- **Raw artefacts** — Lighthouse JSONs and a baseline screenshot saved alongside the report so future audits can diff.

Bands: **≥ 85 publish-ready · 70–84 targeted fixes · < 70 rework.**

---

## Install

Drop the two pieces into your project (or your user-level Claude Code config):

```bash
git clone https://github.com/notoriousmaps95/Page-Performance.git
cd Page-Performance

# 1. Copy the module folder into your project root
cp -R page-performance /path/to/your/project/

# 2. Copy the subagent definition into your Claude Code agents folder
#    Project-scoped:
mkdir -p /path/to/your/project/.claude/agents
cp .claude/agents/page-performance.md /path/to/your/project/.claude/agents/

#    OR user-scoped (available from any project):
mkdir -p ~/.claude/agents
cp .claude/agents/page-performance.md ~/.claude/agents/
```

That's it. Claude Code auto-discovers the agent once the file is in place.

---

## Use

From inside the host project, in a Claude Code session:

```
Use the page-performance agent with url=https://example.com/page
```

Or invoke directly:

```
Agent({
  subagent_type: "page-performance",
  prompt: "url=https://example.com/page"
})
```

The agent runs **Phase 1 (baseline measurement)** — Lighthouse mobile + desktop, CrUX/GSC field pull, WebPageTest filmstrip, page weight, full-page screenshot — and **stops** with the baseline table. Reply with confirmation (or "skip to audit") and it walks Phases 2–10 + emits the scorecard in one continuation turn.

### Inputs

| Input | Required | Notes |
|---|---|---|
| `url` | yes | Full URL of the page being audited |
| `mode` | no | `quick` (lab-only, skip CrUX) · `full` (default — lab + field + WebPageTest) |
| `competitors` | no | Comma-separated URLs to benchmark against (max 3) |

### Output

Written to `page-performance/reports/<page-slug>/`:

| File | Purpose |
|---|---|
| `audit-<YYYY-MM-DD>.md` | The `/100` audit report (scorecard + top-5 + Next Actions) |
| `baseline-<YYYY-MM-DD>.png` | Phase 1 full-page screenshot |
| `lighthouse-mobile-<YYYY-MM-DD>.json` | Raw Lighthouse mobile report |
| `lighthouse-desktop-<YYYY-MM-DD>.json` | Raw Lighthouse desktop report |
| `notes.md` | Free-form findings, edge cases, follow-ups |

Page slug derives from the URL: `https://example.com/services/hvac` → `example-com-services-hvac`.

---

## The 10 phases

| # | Phase | What it checks |
|---|---|---|
| 1 | Baseline measurement | Lighthouse mobile + desktop, CrUX/GSC field data, WebPageTest filmstrip, page weight, TTFB, full-page screenshot |
| 2 | HTML payload | Size (`<450KB`), validation, encoding, single `<h1>` / `<title>` / canonical |
| 3 | Compression + protocol | Brotli, HTTP/2 or HTTP/3, server-push audit |
| 4 | Resource prioritisation | `preconnect`, `preload`, `defer`, `async`, `fetchpriority` |
| 5 | Image optimisation | AVIF/WebP, `srcset`, explicit dimensions, LCP preload, lazy-loading, EXIF stripping |
| 6 | Font optimisation | Variable fonts, `font-display: swap`, icon-font subsetting (with the equality assertion to prevent missing-icons regressions) |
| 7 | CSS + JS efficiency | Segmentation, unused-code removal, cache headers, minification (≤25KB / ≤3 files) |
| 8 | DOM size + semantic HTML | <1,500 nodes, correct landmark structure, sequential heading nesting |
| 9 | Caching / PWA + accessibility | `Cache-Control`, `ETag`, service worker, manifest, alt/labels/ARIA/contrast |
| 10 | CWV diagnosis + structured-data link | LCP/INP/CLS root-cause, confirm declared schema `image` matches the rendered LCP element |

---

## Scoring

`/100` across 9 weighted categories:

| # | Category | Weight |
|---|---|---:|
| 1 | Core Web Vitals (LCP · INP · CLS, field-data preferred) | 25 |
| 2 | Image optimisation | 15 |
| 3 | Font + CSS + JS efficiency | 15 |
| 4 | Resource prioritisation | 10 |
| 5 | HTML payload size & validation | 10 |
| 6 | Caching · compression · protocol | 10 |
| 7 | DOM size & semantic HTML | 8 |
| 8 | Accessibility signals | 4 |
| 9 | Structured-data ↔ performance link | 3 |
| | **TOTAL** | **100** |

Full rubric in [`page-performance/rules/scoring.md`](page-performance/rules/scoring.md).

---

## What this does NOT cover

- **Full site audit** — one page at a time. For multi-page benchmarks, run it once per template (homepage, service, about, etc.).
- **Implementing fixes** — produces the audit + recommendations; a dev (or separate agent) executes.
- **CMS-specific deployment** — recommendations are technology-agnostic.
- **A/B testing or CRO** — performance only.
- **Crawl-budget / log-file analysis** — different module.
- **Multi-locale performance comparison** — v1 single-locale only.

Keep each module narrow. One job, one answer.

---

## Repo layout

```
Page-Performance/
├── README.md                              this file
├── LICENSE
├── .claude/
│   └── agents/
│       └── page-performance.md            the subagent definition — copy to ~/.claude/agents/ or <project>/.claude/agents/
└── page-performance/                      drop into your project root
    ├── CLAUDE.md                          module-level invocation + layout doc
    ├── rules/
    │   ├── workflow.md                    10-phase order, dependencies, stop points
    │   ├── scoring.md                     /100 rubric (9 categories, bands, evidence rules)
    │   ├── performance-budgets.md         numeric budgets (HTML, CSS/JS, DOM, CWV, caching)
    │   └── semantic-and-a11y.md           landmark structure, headings, ARIA, contrast, CLS
    ├── sops/
    │   └── optimise-page-performance.md   the full 10-phase SOP (Goal · Steps · Verification · Pitfalls)
    ├── templates/
    │   └── audit-report.md                scorecard skeleton — copied into reports/<slug>/
    ├── samples/                           reserved — case studies from real audits
    └── reports/                           per-page output folders land here
```

---

## Credits

Built on the framework from Koray Tuğberk GÜBÜR's [Holistic SEO Research Study (Encazip case)](https://www.holisticseo.digital/seo-research-study/holistic-seo). The phase order, budgets, and the icon-font subsetting pitfall all trace back to that study.

---

## License

MIT — see [LICENSE](LICENSE).
