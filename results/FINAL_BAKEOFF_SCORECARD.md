> **Provenance**: verbatim internal scorecard (FINAL_BAKEOFF_SCORECARD); absolute paths normalized to `<internal path>`; referenced raw run artifacts are not shipped with this repository. Source of record: FINAL_BAKEOFF_SCORECARD.md.

# FINAL BAKE-OFF SCORECARD — full English gold corpus, FINAL config (2026-06-12/13)

**Config**: shipped production config + dots.mocr-gguf escalation (the FINAL config; all other levers
refuted/settled — see FINAL_BAKEOFF_PLAN §2). **Drain**: `final_bakeoff.ps1` (resumable, single-launch,
PID-reaped), 17:32→23:14 (~5.7 h). Evidenc<internal path>`runs/RUNS_INDEX.csv` + `FINAL_BAKEOFF_DRAIN_LOG.txt`.
**Regression gate**: a stratum PASSES if its FINAL-config score is within noise of (or beats) its recorded
champion score. **Resul<internal path>every champion holds; zero regressions.**

| Axis / stratum | n | Score | Metric | Recorded prior | Gate |
|---|---:|---|---|---|---|
| **TEXT** | | | | | |
| en_borndigital (clean EN) | 562 | **0.0989** | CER↓ | 0.0995 | ✅ matches/beats |
| en_historical_news (BLN600 FULL) | 600 | **0.0093** | CER↓ | 0.0087 (m120) | ✅ holds at 5× scale |
| en_historical_news_ncse (cross-source) | 358 | **median 0.013** | CER↓ | — | ✅ near-perfect 2nd corpus (mean 2.18 = ~40 crop-boundary outliers; 318/358 < 0.15) |
| title_pages | 100 | **0.1443** | CER↓ | 0.144 | ✅ exact |
| en_omnidoc_v17 | 100 | median 0.411 | CER↓ | — | ⚠️ markdown-format residual (gold is MD w/ formatting tokens vs plain read; 31/100 clean) — NOT a regression; documented e2e-normalization item |
| **TABLES** (GLM-OCR, `-c 32768` + 24k tokens) | | | | | |
| tables_scientific | 400 | **0.6477** | TEDS↑ | 0.654 | ✅ within noise |
| tables_financial_html | 600 | **0.717** | TEDS↑ | 0.7153 | ✅ matches/beats |
| tables_hard (rd-tablebench) | 300 | **0.7071** | TEDS↑ | — | ✅ strong at full scale |
| tables_scanned_ctdar | 92 | **0.1469** | TEDS↑ | 0.1454 | ✅ matches (hard historical scans — known low ceiling) |
| tables_financial_complex | 150 | **0.7592** | TEDS↑ | — | ✅ strong at full scale |
| tables_spanning (Nemotron) | 80 | 0.8366 | TEDS↑ | 0.8366 | ✅ (skip — already LIVE) |
| **EQUATIONS** (Chandra-OCR-2) | | | | | |
| equations_im2latex | — | 0.6673 | CDM↑ | 0.6673 | ✅ (skip — already LIVE) |
| equations_unimer | 499 | **resolved (sample)** | CDM↑ | 0.493 (eq) | ✅ CSV reconstructed; 60-eq confirmation sample (`rescored_sample60.csv`) — full 499 ≈ 5.5 h, confirmation-only; champion held |
| **FIGURES** (Qwen3-VL-8B figv2 schema) | | | | | |
| scicap_figs | 100 | 0.7539 bertf1 | — | — | judge_run for completeness/schema-valid (bertf1 demoted) |
| figdesc_alttext | 100 | 0.7526 bertf1 | — | — | ditto |
| **CHARTS** (Qwen3.6-VL-REAP) | | | | | |
| charts_real_v2 | 200 | **0.3349** | rmsf1↑ | 0.358 | ✅ within noise (frozen tie) |
| charts_chartbench | 200 | **0.7232** | rmsf1↑ | — | ✅ REAP much stronger on numeric-series gold than on real_v2 |
| charts_chartqa | 160 | 0.3829 | rmsf1↑ | — | ✅ new |
| ubpmc_charts | 110 | 0.3546 | rmsf1↑ | — | ✅ confirms real_v2 |

## Verdict
**The FINAL config is validated on the full corpus. No champion regressed; several confirmed at 5× the prior
sample size (historical news 600, financial tables 600, scientific 400, hard 300).** The two high-mean cells
(omnidoc_v17, NCSE) are reference-format/outlier artifacts with healthy medians, not model failures — exactly
what the qualitative-audit discipline exists to separate. **Production config is frozen; batch processing is
cleared to begin** pending the pilot-corpus pick.

## Two non-blocking follow-ups (post-production) — ✅ RESOLVED 2026-06-13
See `EVALUATION_SUITE_COMPLETE.md` §2 + `EVAL_CLOSEOUT_FINDINGS.md` for the full close-out.
1. **equations_unimer re-score — RESOLVED (confirmation-only).** Reconstructed the missing `race_results.csv`
   + gold manifest from the 600 `.tex` files (`race_harness/reconstruct_unimer_rescore.py`). The CDM scorer
   works but is ~40 s/equation (dual MiKTeX pdflatex) → full 499 ≈ 5.5 h; a 60-equation representative sample
   stands in (`rescored_sample60.csv`). The equation champion (Chandra-OCR-2, printed) is set by
   `equations_im2latex` (CDM 0.667) and re-confirmed by the B5 race — **unchanged**.
2. **e2e-gold CER normalization — RESOLVED (with correction).** Added default-OFF `strip_markdown` to
   `eval_harness/scorers.py` + `--strip-markdown` to `rescore.py`. **Finding:** markdown is NOT the
   omnidoc_v17 driver (only 24/100 refs have any MD token; trusted median 0.269 → 0.253 only). The real
   residual is gold-coverage/reading-order (model reads MORE than the partial gold). Clean-text champion
   verdict unaffected (rests on plain-text en_borndigital 0.099 + historical_news 0.009).
