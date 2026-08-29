# Metrics, champion selection, and the n-statement policy

Every number in this repository follows these rules. If a document here violates one, it is a bug.

## Metrics

| Metric | Full name | Measures | Direction | Used for |
|---|---|---|---|---|
| **CER** | Character Error Rate | edit distance between extraction and gold, normalized by gold length | ↓ lower is better | body text (born-digital, scanned, historical) |
| **TEDS** | Tree-Edit-Distance-based Similarity | structural similarity of table trees (cells + merges + headers) | ↑ higher is better | tables (HTML/financial/scientific) |
| **CDM** | Competition-Distance Metric | rendered-equation similarity (compile + image distance) | ↑ | equations (printed, handwritten) |
| **RMS-F1** | — | F1 over chart data series (values + labels) recovered as data | ↑ | charts→data |
| **order-BLEU** | — | BLEU over reading-order sequence vs gold order | ↑ | layout/reading order |

Notes: CER means are outlier-sensitive on crop-boundary gold (see the NCSE triage in
EVALUATION_SUITE_COMPLETE.md — medians reported alongside means where that matters). bertf1 was
retired as a primary figure metric (kept for completeness rows only).

## Champion selection (the rule that keeps leaderboards honest)

The champion for a content type is selected as:

1. For each **gold page**, take each model's **best (model, prompt) result on that page**.
2. Aggregate per model over pages (mean CER / TEDS / CDM per stratum).
3. The champion is the best aggregate **per stratum**, then confirmed (or routed) per
   content × era cell.

**Never** a grand mean over all prompts (a catastrophic-loop prompt run can inflate a model's mean
by 40× — observed: one model's mean was 2.41 while its best-per-page aggregate was 0.055). The
selection script ships in the lineage docs' methodology sections.

## n-statement policy

1. Every score names its gold set and **n** (pages or items).
2. Strata confirmed at larger n supersede smaller-n priors, and both are shown when the bake-off
   re-ran them (e.g. historical news: 0.0087 at n=120 → 0.0093 held at n=600).
3. Where a full re-score was impractical, the stand-in sample is named as a sample with its size
   (e.g. the 60-equation confirmation sample) and is never presented as the full set.
4. Indicative numbers (order-of-magnitude, development-use) are labeled *indicative* — none of the
   scorecard tables in this repo fall in that class; the label exists so the distinction is
   enforceable.

## What is not claimed

- No single headline accuracy number for "documents" — quality is per content type, per stratum.
- No public benchmark leaderboard claims: gold sets here are the program's own (public-domain and
  licensed sources, assembled for these races); where a public benchmark was used it is named
  (OmniDocBench, ICDAR-2019 post-OCR, ChartQA, im2latex, HME100K, CTdar, SciCap).
- Gold pages and their transcriptions are not redistributed in this repository (source licensing);
  the scorecards record what was measured and how.
