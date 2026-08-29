> **Provenance**: verbatim internal scorecard (DEFINITIVE_ROUTING_TABLE_20260612); absolute paths normalized to `<internal path>`; referenced raw run artifacts are not shipped with this repository. Source of record: DEFINITIVE_ROUTING_TABLE_20260612.md.

# THE DEFINITIVE ROUTING TABLE — English Extraction Engine (2026-06-12)

**This is the deliverable the program existed to produce** (ENGLISH_ENGINE_MASTER_PLAN §5.7): content × era →
model → config → verifier → escalation, every cell evidence-cited. Statu<internal path>**measured + bake-off-confirmed;
production adoption pends the P7.1 SHIP review** (`SHIP_REVIEW_20260612.md`). Bake-off evidenc<internal path>`race_harness/results_bakeoff/` (V1 ≥ V0 on every fairly-measured axis, 60-page held-out set, 40/60 UNSEEN).

| Content × era | Route to | Config | Verify (CPU) | Escalate | Evidence |
|---|---|---|---|---|---|
| **Clean / born-digital EN text** | text-layer (PyMuPDF); scanned → **dots.ocr** | Q8, `-fa`, KV q8, img-min-tokens 1024, greedy | CE-gate inert solo (correct) | olmOCR-2-7B on content-failure gate | dots 0.0995 CER (n=459) |
| **Historical / degraded EN text** ★ | **dots.ocr** | same | — | **dots.mocr** (native) on abstention/empty | 0.0087 m120 / **0.0097 held-out UNSEEN**; dots.mocr 0.0118 full-coverage |
| **Handwritten EN** | olmOCR-2-7B | Q8, greedy | — | — | 0.055 IAM (print models generalize) |
| **Tables — regular/scientific** | GLM-OCR | **F16 pref**, whole-page | C1 cross-foot → review queue | Nemotron-Parse on spanning-detector fire (default-OFF, gated) | GLM 0.8345 omnidoc / 0.7153 fintab |
| **Tables — historical/statistical (dense archival)** ★ | **GLM-OCR** | **`-c 32768` + max_tokens ≥24k** (engine cap fix = ship-pair) | C1 cross-foot + cell-recall report | flag low-recall pages to review | 0.6018 TEDS + 0.877 cell-recall (v3); bake-off 0.6249 > 0.6031 |
| **Tables — TOC/leader-dot layout** ★ | **REROUTE → body text** (`toc_triage.is_toc_layout`, CPU pre-pass) | threshold 0.5 | — | exhibit-index residual → C1 queue | P 1.00 / R 1.00 on labeled cluster; OTSL + caps both refuted on this cluster |
| **Equations — printed** | Chandra-OCR-2 | Q8, eq-crop, `<think>` strip | (A²R² render-gate built, value-run residual) | — | 0.493 CDM; voting rejected; 2B challenger rejected 0.2032 |
| **Equations — handwritten** | Uni-MuMER (native) | bf16 SDPA | — | — | 0.835/0.821 CDM (LIVE-confirmed 06-09) |
| **Figures — description** ★ | **figdesc-v2 schema prompt** on gemma-4-E4B-figv2 *or* Qwen3-VL-8B-figv2 (user pick) | grammar-OFF; instruction-before-image | `judge_run.py` (schema-valid + completeness) per batch | null-over-guess enforced by prompt | 96/100+0.893 vs 99/100+0.892; bertf1 retired as primary |
| **Figures — chart→data** | Qwen3.6-VL-REAP (frozen tie) | thinking-off, GBNF-JSON | series↔description agreement flag | CV-ensemble only line/single-bar (C4) | 0.358 rmsf1; Granite candidate gated |
| **Layout / reading order** | DocLayout-YOLO + XY-Cut++ | CPU | — | PP-DocLayout opt-in math-dense | order_bleu 0.44 |
| **Cyrillic / Arabic / Swedish** | FROZEN per 2026-06-11 de-scope | dots.ocr Cyrillic; Arabic OPEN | — | — | unchanged |

★ = new/changed this campaign (2026-06-11/12).

> **2026-06-13 close-out confirmations** (`EVALUATION_SUITE_COMPLETE.md`): the **Equations — handwritten**
> route is now wired in code — `dispatch.py` `equation_handwritten` → Uni-MuMER (native), **default-OFF /
> fail-safe to printed Chandra**, gated on cfg `equations_are_handwritten` / `doc_hint="handwritten_math"`
> (deciding rac<internal path>Uni-MuMER CDM 0.752 vs Chandra 0.520, +45% on `eq_handwritten_hme100k`). The **chart→data**
> scanned-transfer gate resolved **HOLD REAP** (REAP `charts_scanned` 0.394 vs Granite 0.447 on scanned-synthetic,
> but TIE on real UB-PMC; REAP lighter gguf incumbent). **Reading order** XY-Cut++ adopt via opt-in flag confirmed
> (+89% reproduced; `readorder_0041` accepted). All three are user-gated switches — no default flipped.

## Throughput & serving (the production recipe, SHIP-gated)
**`-np 4 -c 65536` + `-fa` + KV q8_0 + greedy/seed + restart-every-100-pages + diff-gate canary** →
25.3 pages/min fresh / ~20 sustained (~30k pages/day) at 8.2 GB VRAM. Slots verified corruption-free
(greedy diff-gate 0 violations @ np1/2/4). Server aging ~52%/100 reqs ⇒ the restart rule. Multi-instance
and vLLM rung<internal path>NOT needed (slots won). Evidenc<internal path>`THROUGHPUT_AND_TRIAGE_RESULTS_20260612.md`.

## Cost (the thesis, quantified)
Local **$0.21–0.94/1k pages** (full pipeline, marginal) vs cloud agent extraction **~$13/1k** (est., cross-checked
against campaign cost notes) ⇒ **~14–60× cheaper**, worst-case-vs-best-case still ≥3×. Plus subscription
relief. `COST_MODEL_20260612.md`.

## Bake-off summary (held-out, V0 frozen vs V1 adoptions vs V2 +verify)
| Route | V0 | V1 | V2 | Read |
|---|---|---|---|---|
| historical_text (20 UNSEEN) | 0.0097 CER | 0.0097 | = | champion generalizes; both variants route dots |
| text (20 UNSEEN e2e slices) | 0.3316 | 0.3316 | = | identical (same route); absolute number is a GT-format mismatch in the new e2e gold — normalization task queued, not a model verdict |
| table (5, confirmation) | 0.6031 | **0.6249** | 0.6249 + queue | caps help; cross-foot clean on clean tables |
| figure (15 semi-held-out) | 0.8491 bertf1 | (schema JSON — bertf1 inapplicable) | = | judge metri<internal path>0.893 completeness on decision slice; bertf1 retired |
**Verdic<internal path>adopt V1 routing + V2's CPU verification layer** — no held-out regression anywhere, two gains.

## Residual (open, none blocking SHIP)
e2e-gold CER normalization (cc_ocr/ocrbench references) · A²R² value-run · post-correction harm boundary ·
logprob char-fusion (low expected value per C3) · OTSL on dense REGULAR tables · np8 + aging-under-np probes ·
overlap rung on real PDFs · T7 mini-run re-run · llama.cpp upgrade (DeepSeek-OCR-2) · NCSE cross-source race.

*Every number traces to runs/RUNS_INDEX.csv or results_bakeoff/. Production unchanged until SHIP review.*
