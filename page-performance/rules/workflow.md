# Workflow — 10-phase audit order

The page-performance audit walks 10 phases in fixed order. Each phase's verification feeds the next phase's diagnosis. Do not reorder. Skipping is allowed (with justification in the report) but reordering wastes a measurement run.

> Full Goal / Steps / Verification / Pitfalls for every phase lives in [`sops/optimise-page-performance.md`](../sops/optimise-page-performance.md). This file is the **conductor's view** — order, dependencies, stop points.

---

## Phase order

| # | Phase | Why this position |
|---|---|---|
| 1 | Baseline measurement | Establishes the frozen "before" snapshot. Every later finding is measured against this. |
| 2 | HTML optimisation | Smallest unit first — if HTML is invalid or oversized, every later phase compounds the cost. |
| 3 | Compression & protocol | Transport layer — gates byte-cost of everything HTML/CSS/JS/font/image delivers. |
| 4 | Resource prioritisation | Decides what the browser fetches *first*. Has to come before image/font/CSS optimisation so the LCP candidate is known. |
| 5 | Image optimisation | LCP candidate is usually an image — bytes + dimensions + format choice dominate field LCP scores. |
| 6 | Font optimisation | FOIT/FOUT and icon-font payload both affect LCP and CLS. Audit after images so subsetting decisions know what the hero font is. |
| 7 | CSS & JS efficiency | Render-blocking + unused-code review. Sized after fonts because font CSS often inflates the CSS budget. |
| 8 | DOM size & semantic HTML | Layout cost. Inflated DOM compounds the cost of every paint, so audit it once HTML/CSS/JS are settled. |
| 9 | Caching / PWA / service worker | Repeat-visit performance. Worthless if first-visit (Phases 1–8) is broken — audit last. |
| 10 | Accessibility + structured-data ↔ performance | Cross-cuts everything above; runs last so all rendered output is final before checking semantic + schema fidelity. |

---

## Stop points

Two stop points in a normal run:

### Stop A — after Phase 1

After capturing baseline metrics, **stop** and present the user with a summary table. Wait for confirmation before walking Phases 2–10.

Why: the baseline numbers often reveal which phases will dominate the fix list. The user may want to redirect (e.g. "the LCP is fine, focus on INP" → spend extra time on Phase 7 JS efficiency).

Skip Stop A only when invoked in autonomous mode (no live user).

### Stop B — after Phase 10

After filling the scorecard and top-5, **stop**. The agent does not implement fixes.

---

## Dependency graph

```
Phase 1 (baseline) ─────────────────────────────────┐
                                                    │
Phase 2 (HTML) ────────────────┐                    │
                               │                    │
Phase 3 (compression) ─────────┤                    │
                               ├──→ Phase 7 (CSS/JS efficiency)
Phase 4 (prioritisation) ──┐   │                    │
                           │   │                    │
Phase 5 (images) ──────────┤   │                    │
                           │   │                    │
Phase 6 (fonts) ───────────┘   │                    │
                               │                    │
                               ├──→ Phase 8 (DOM + semantic HTML)
                               │
                               └──→ Phase 9 (caching / PWA)
                                                    │
                                                    └──→ Phase 10 (a11y + schema link)
                                                                  │
                                                                  └──→ Scorecard
```

- Phases 2–6 are **measurement-heavy** — each one needs the live page loaded with cache disabled.
- Phases 7–10 are **synthesis-heavy** — they reuse the network/coverage/DOM traces from earlier phases.
- Phase 1's screenshot is referenced by Phase 10 (visual diff check after a11y fixes) — don't skip the screenshot.

---

## When to skip a phase

| Phase | Skip when | Note in report |
|---|---|---|
| 1 — Baseline | Never. Even a 30-second baseline is required. | — |
| 2 — HTML | HTML is dynamically rendered by a SPA framework and source HTML is < 5 KB | "SPA shell — HTML budget audit deferred to render output" |
| 3 — Compression | Page served from a known-good CDN (Cloudflare, Vercel) AND `curl -sIL` confirms `br` + `h2`/`h3` in < 30s | "Brotli + HTTP/2 confirmed in baseline; deep audit deferred" |
| 4 — Prioritisation | Never if LCP > 2.5s. | — |
| 5 — Images | Page has zero `<img>` tags (text-only page) | "No images on page" |
| 6 — Fonts | Page uses only system fonts (no `@font-face`) | "System fonts only" |
| 7 — CSS/JS | Page has < 5 KB CSS + JS total | "CSS/JS already under budget — audit deferred" |
| 8 — DOM | DOM < 500 nodes per baseline | "DOM well under budget" |
| 9 — Caching | Audit is for a fresh site without repeat-visit traffic | "First-visit focus — defer caching audit" |
| 10 — A11y | Never. Lighthouse a11y score is always captured. | — |

A skipped phase still gets a row in the scorecard, with the skip reason in the notes column. Score the row as if budgets were met (assume green) and flag the assumption.

---

## Cross-references

| Need | File |
|---|---|
| Full Goal / Steps / Verification / Pitfalls per phase | [`sops/optimise-page-performance.md`](../sops/optimise-page-performance.md) |
| `/100` rubric, weights, evidence requirements | [`rules/scoring.md`](scoring.md) |
| Numeric budgets per phase | [`rules/performance-budgets.md`](performance-budgets.md) |
| Semantic HTML + A11y rule (Phases 8 and 10) | [`rules/semantic-and-a11y.md`](semantic-and-a11y.md) |
| Final scorecard skeleton | [`templates/audit-report.md`](../templates/audit-report.md) |
