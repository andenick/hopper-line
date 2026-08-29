# Hopper Line — Local Multi-Model Document Extraction, Evidence-First

**An offline PDF→structured-data engine for one consumer GPU (RTX 5090, 8–24 GB VRAM class): per-region model routing, where every routing decision is backed by a measured race on real gold, and the full quality evidence ships with the method.**

> **What this repository is**: the **method specification plus the complete empirical quality record**
> of the Hopper Line v2 engine ("HL2") — routing tables, champion scorecards, evaluation close-outs —
> distilled from a production research program (2026-05 → 2026-06). The engine itself runs on local
> GPU infrastructure with several open-weight models; this repo ships the *method* and the
> *evidence*, plus pointers for rebuilding, rather than a turnkey installer (see
> [method/HL2_PROTOCOL.md](method/HL2_PROTOCOL.md) §0).
>
> **On the numbers**: unlike most model write-ups, every number here is a **measured result on a named
> gold set with its n stated** (CER/TEDS/CDM/RMS-F1 — definitions in
> [results/METRICS.md](results/METRICS.md)). Where a number is indicative rather than measured, it
> says so. Where an evaluation had a flaw, the correction ships with it (see the omnidoc_v17 and
> NCSE close-outs — reported, not hidden).

---

## The one idea: route per region, not per document

A scanned statistical yearbook is four different problems on one page: body text, a dense table,
an equation, a chart. HL2 runs a layout pass, then routes **each region** to the content-type
champion model — with a cheap default and confidence-gated escalation, so the expensive model only
sees the regions that need it:

| Region type | Champion | Measured (gold, n) |
|---|---|---|
| Born-digital text | text-layer bypass (no OCR at all) | — |
| Clean/historical EN text | **dots.ocr** | CER **0.0093** (historical news, n=600) |
| Degraded EN text | **dots.mocr** (escalation) | CER 0.0118 (full coverage) |
| Tables — scientific/financial/dense archival | **GLM-OCR** | TEDS **0.65–0.76** (n=400–600/stratum) |
| Printed equations | **Chandra-OCR-2** | CDM **0.667** (im2latex, n=600) |
| Handwritten equations | **Uni-MuMER** (opt-in) | CDM 0.752 (n=100) |
| Figure descriptions | **Qwen3-VL-8B** (schema prompt) | schema-valid 0.89+ (n=200) |
| Charts → data | **Qwen3.6-VL-REAP** | RMS-F1 0.33–0.72 (n=110–200/stratum) |
| Layout / reading order | DocLayout-YOLO + XY-Cut(++) | coverage 0.937; order-BLEU +89% (opt-in) |

Full table with configs, verifiers, escalation paths, and per-cell evidence citations:
**[method/ROUTING.md](method/ROUTING.md)**.

## The evidence (all shipped, all n-stated)

- **[results/FINAL_BAKEOFF_SCORECARD.md](results/FINAL_BAKEOFF_SCORECARD.md)** — the frozen
  production config validated on the full gold corpus: **17 strata, ~3,400 rows, zero regressions**;
  champions confirmed at up to 5× their original sample size.
- **[results/EVALUATION_SUITE_COMPLETE.md](results/EVALUATION_SUITE_COMPLETE.md)** — per-content-type
  close-out: every champion, its gold, its metric, its verdict — plus two honest corrections
  (a markdown-normalization hypothesis refuted; a cross-corpus outlier triaged to crop-boundary
  artifacts, median exonerated).
- **[results/FROZEN_CHAMPIONS.md](results/FROZEN_CHAMPIONS.md)** + **[results/THUNDERDOME_V3_LINEAGE.md](results/THUNDERDOME_V3_LINEAGE.md)**
  — the earlier evidence generations: the multi-engine "thunderdome" races (2,214 runs) that seeded
  the routing, and the frozen champion matrices including the filename-renamer consensus panel.
- **[results/METRICS.md](results/METRICS.md)** — what CER, TEDS, CDM, and RMS-F1 mean here, how
  champions are selected (best model×prompt per page, then aggregate — never a grand mean), and the
  n-statement policy.

## Why it matters

- **Local and cheap**: $0.21–0.94 per 1,000 pages marginal (electricity), ~20–25 pages/minute
  sustained at ~8 GB VRAM — versus cloud-agent extraction orders of magnitude more.
- **No silent quality loss**: cross-model confidence gating (Spearman −0.98 vs CER on degraded
  pages), cross-foot verification on tables, review queues for low-confidence regions, and a
  "null over guess" policy for figure descriptions.
- **Evidence discipline**: nothing ships without a race on real gold; synthetic evals are
  stress-tests only; refuted candidates are recorded next to the winners.

## Relationship to other work

Hopper Line is a **local GPU engine**, distinct from the agent-reading protocol family (HDARP) —
different failure taxonomy, different cost profile; see method/HL2_PROTOCOL.md §Scope. The two are
complementary: protocol-driven agent reading for hard documents in the cloud, evidence-routed local
extraction at scale.

## Repository layout

```
method/    HL2_PROTOCOL.md (the method), ROUTING.md (the definitive routing table)
results/   scorecards, close-outs, champion lineages, METRICS.md
```

License: MIT (see LICENSE). Citation: see CITATION.cff.
