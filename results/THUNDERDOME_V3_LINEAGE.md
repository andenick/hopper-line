> **Provenance**: verbatim internal scorecard (B4_EXTRACTOR_CHAMPIONS); absolute paths normalized to `<internal path>`; referenced raw run artifacts are not shipped with this repository. Source of record: B4_EXTRACTOR_CHAMPIONS.md.

# Phase B4 — Frozen Per-Content-Type Extractor Champion Matrix

**Date**: 2026-05-23
**Author**: Hopper Line battery analysis (B4)
**Inputs (all real, on disk, no GPU re-run)**:
- Results DB (internal, since archived): the `runs` table (2214 rows; 57 with gold CER, 15 with TEDS).
- Canonical leaderboard: the Thunderdome v3 FINAL report (internal, 2026-05-22); its rankings are reproduced below.
- Cell map: a labeled corpus manifest (page id to language / content type / scan quality / era).
- Gold: 6 HTML pages.

**Scope**: This is pure mining of existing results into a frozen per-(language × content_type) champion matrix and a reconciliation with the Line's routing config. No model was re-run. No gold or CER number was fabricated — every figure below traces to a row in `runs` or a gold file.

---

## 1. Canonical Thunderdome v3 leaderboard (located & summarized)

**Source of record**: `THUNDERDOME_V3_REPORT.md` (FINAL). Headline ranking<internal path>- **Accuracy (gold CER ↓, lower better)**: **Qwen3.6-VL-REAP 0.055** #1 · GLM-4.6V-Flash 0.096 · dots.ocr 0.098 · Chandra-OCR-2 0.119 · GLM-OCR 0.137 · olmOCR-2-7B 0.164 · gemma-26B 0.199 · Qianfan 0.253 · gemma-31B 0.254 · gemma-E4B 0.435. The SOTA vLLM model PaddleOCR-VL-1.5 scored 0.079 (2nd-best) — **the native roster's Qwen-REAP beat the OmniDocBench #1**.
- **Per-content-type champions reported**: prose → **Qwen-REAP**; tables → **dots.ocr** (TEDS 1.00); equations → **Chandra-OCR-2** (CDM 0.82); forms → dots / GLM-4.6V; throughput/reliability → dots.ocr (cleanest, rep 0.021) & GLM-OCR (986 pg/hr).
- **Production verdict**: stay native llama.cpp; default `dots.ocr` + escalate `Qwen3.6-VL-REAP` + content-type routing.

**Methodology I reused (and verified)**: the `0.055` figure is Qwen-REAP's **mean of best-per-page CER over the 3 high-confidence gold pages** (031_instructions form, Capitalism_p051 form, Capitalism_p651 equation), using its routed prompt `p01_html_strict` on all three. I reproduced it exactly from `runs`: Qwen 0.0550, dots 0.0978, GLM-4.6V 0.0706, gemma-26B 0.1956 — matching the report's leaderboard order. This confirms the champion-selection rule is **best (model, prompt_id) per page, then aggregate** — NOT a grand mean over all prompts (which would inflate Qwen to ~2.41 via the p03/p00 catastrophic-loop pages).

---

## 2. Per-(language × content_type) extractor champion matrix (computed from `runs`)

**Gold coverage is 6 pages** mapping to 5 cells. Only the 3 **born-digital / clean-scan English** gold pages have usable CER (HTML gold aligns with output). The SV/prose and RU/cyrillic gold HTML does **not** align with model output format → every model's CER on them is 2–29 (a format/gold mismatch, NOT model error); those are the `score_v3.PENDING` pages and are treated as **ref-free only** here. Ranking rule per cel<internal path>best (model, prompt_id) by the cell-appropriate metric (CER for text/form; **TEDS for tables**; structure for equations). n = gold pages in the cell.

| Cell (lang/ct) | Gold? | n | Metric | **Champion** (best prompt) | Champion score | Runner-up + margin | CI / confidence |
|---|---|---|---|---|---|---|---|
| **EN/form** | gold | 2 | CER↓ | **Qwen3.6-VL-REAP** (p01_html_strict) | CER 0.000 (p051), 0.114 (031) | GLM-4.6V tie 0.000/0.132; dots 0.009/0.200 | Qwen & GLM tie at 0.000 on the clean form page; Qwen wins the born-digital 031 form (0.114 vs 0.132). 2 pages — thin. |
| **EN/equation** | gold | 1 | CER↓ | **Qwen3.6-VL-REAP** (p01_html_strict) | CER 0.0506 | GLM-4.6V 0.0801 (Δ0.029); dots 0.0845 (Δ0.034) | **n=1** — single page. CI collapses to point estimate. Chandra was NOT CER-scored here (see §5). |
| **EN/table** | gold | 1 | **TEDS↑** (+CER) | **dots.ocr** (p00_base) | **TEDS 1.000** / CER 0.188 | Qwen p03 TEDS 0.967 (Δ0.033); GLM-4.6V p03 0.625 | **n=1**. dots is perfect on structure; Qwen edges on raw CER (0.183 vs 0.188) but loses TEDS by 0.41 at p00 and only ties at p03. Tables judged on TEDS → dots. |
| **SV/prose** | no gold (PENDING) | 0 usable | ref-free | **dots.ocr / Qianfan / olmOCR** (rep 0.000, 0 empty) | — | — | **NO GOLD.** All CER 2–29 = gold/format mismatch, not accuracy. Ref-fre<internal path>dots/Qianfan/olmOCR cleanest (rep 0.000, n=5); GLM-OCR & gemma-E4B loop (rep 0.20–0.28). |
| **RU/cyrillic** | no gold (PENDING) | 0 usable | ref-free (+TEDS hint) | **dots.ocr** (rep 0.034, lowest) | — | olmOCR rep 0.083; Qianfan 0.114 | **NO GOLD.** 57 pages all ok%=100, 0 empty. dots cleanest; GLM-OCR & PaddleOCR loop badly (rep 0.32–0.47). Only TEDS hints exist (Chandra 0.29, GLM-4.6V 0.02) and are unreliable vs unverified gold. |
| EN/prose | no gold (saturated) | 0 | ref-free | **gemma-31B / olmOCR / Qwen** (rep ≤0.005) | — | — | **NO GOLD on the prose cell directly** (the gold "prose" signal lives in the form/equation pages). dots.ocr empties 13% here (87% ok) — a known weak cell for dots. |
| EN/equation (ref-free) | — | 12 | rep | all ≤0.010 except PaddleOCR (0.154) | — | — | Saturated cell — all reliable. |

**Reading the matrix honestly**: only **EN/form, EN/equation, EN/table** are gold-anchored, and two of those rest on a **single page**. Every other cell is ref-free reliability only. The matrix refines — does not overturn — the canonical report.

---

## 3. Single recommended champion per content type (the Line's definitive picks)

The user wants ONE extractor per content type. Evidence-ranked, gold-anchored where possibl<internal path>| Content type | **Definitive champion** | Best prompt | Evidence | Margin | Confidence |
|---|---|---|---|---|---|
| **table** | **dots.ocr** | p00_base (prompt-invariant) | **TEDS 1.000** on the one gold table (Sahr_Inflation), perfect; lowest repetition among reliable producers; canonical report's table champion. | TEDS Δ +0.033 over next-best (Qwen p03 0.967); +0.41 over Qwen p00. | **gold-anchored, n=1** (structure metric is decisive). |
| **equation** | **Qwen3.6-VL-REAP** | p01_html_strict | Lowest gold CER 0.0506 on the one equation page; #1 overall accuracy. | Δ −0.029 CER vs GLM-4.6V; −0.034 vs dots. | **gold-anchored, n=1.** NOT<internal path>the report names **Chandra-OCR-2** for equations on a **CDM** metric that is **NOT in the DB** — see §5. By the DB evidence (CER), Qwen wins; Chandra was not CER-scored on the equation page. |
| **prose / text** | **Qwen3.6-VL-REAP** | p00_base (Latin) | #1 overall gold CER 0.055; report's prose champion. dots.ocr is the reliable default but empties 13% on EN/prose. | dots 0.098, GLM-4.6V 0.071 by aggregate hi-conf CER. | **gold-anchored via the hi-conf set** (form+equation pages stand in for prose accuracy). |
| **form** | **Qwen3.6-VL-REAP** | p01_html_strict | CER 0.000 on the clean form page (tied with GLM-4.6V) AND wins the born-digital 031 form (0.114 vs GLM 0.132, dots 0.200). | Δ −0.018 vs GLM-4.6V on the differentiating born-digital form. | **gold-anchored, n=2.** dots is the safe reliable fallback. |
| **cyrillic** (lang override) | **dots.ocr** | p05_lang_preserve | **NO usable gold.** Ref-fre<internal path>dots cleanest on 57 RU pages (rep 0.034, 0 empty); olmOCR 2nd (0.083). The report flags Cyrillic as the danger zone (consensus 0.39). Current config uses Qianfan (rep 0.114) — defensible but NOT the cleanest. | rep Δ: dots 0.034 < olmOCR 0.083 < Qianfan 0.114. | **ref-free only — no gold.** See §4 mismatch + §5 caveat. |
| **swedish** (lang override) | **dots.ocr** | p05_lang_preserve | **NO usable gold.** Ref-fre<internal path>dots/Qianfan/olmOCR all rep 0.000, 0 empty on 5 SV pages. dots is also the global default → no extra model swap. | tie at rep 0.000 (dots/Qianfan/olmOCR). | **ref-free only — no gold.** |
| **figure** | **__describer__** (sentinel) | n/a | Figures are described, not OCR'd — no change. | — | by design. |

**Overall default / escalation** (matches canonical verdict): **default = dots.ocr** (reliable, fast, perfect tables, cleanest output, produces on every language), **escalate = Qwen3.6-VL-REAP** (gold CER 0.055 #1, wins equation/form/prose). No change.

---

## 4. Reconciliation with the Line's current routing — mismatches

### 4a. `hopperline/pipelines/_common.py` → `CHAMPION_MODELS`

Curren<internal path>```python
CHAMPION_MODELS = {
    "table": "dots.ocr",          # ✅ MATCHES — TEDS 1.00, gold-anchored. KEEP.
    "equation": "Chandra-OCR-2",  # ⚠️ MISMATCH — see below.
    "prose_scan": "dots.ocr",     # ✅ defensible — reliable scanned-prose default. KEEP.
    "cyrillic": "Qianfan-OCR",    # ⚠️ MISMATCH (soft) — see below.
    "figure": "__describer__",    # ✅ KEEP.
}
ESCALATE_MODEL = "Qwen3.6-VL-REAP"  # ✅ MATCHES gold #1. KEEP.
```

**Mismatch 1 — `equation` → Chandra-OCR-2.**
- *Evidence in the DB*: Chandra-OCR-2 was **never CER-scored on the equation gold page** (Capitalism_p651). The only DB evidence on that page ranks **Qwen-REAP 0.0506 < GLM-4.6V 0.0801 < dots 0.0845**. Chandra's CER rows exist only for the PENDING Cyrillic/table/SV pages (all catastrophi<internal path>1.83–4.42, gold mismatch).
- *Why Chandra is in the config*: the canonical report names Chandra the equation champion on a **CDM (Character Detection Matching) proxy = 0.82**, a math-structure metric. That metric is **not persisted in `runs`** (no `cdm_proxy` column populated for the equation page), so it cannot be re-verified here.
- *Verdict*: **defensible but unverifiable from the DB, and contradicted by the one CER number we do have.** Two honest option<internal path>- **(a) Conservative — keep Chandra** (trust the report's CDM finding; equation structure ≠ CER).
  - **(b) Evidence-led — switch to Qwen-REAP** (the only equation metric reproducible from `runs` favors it, and it is the escalation model anyway, simplifying the roster).
- **Recommendation**: switch to **Qwen3.6-VL-REAP** for equation, because (i) it is the only DB-reproducible equation result, (ii) it is already the escalation model so no extra GPU swap, (iii) the Chandra/CDM claim rests on a metric absent from the frozen DB. Flag for the user that this overrides a report claim resting on an unpersisted metric.

  **Proposed edit** (`_common.py`):
  ```python
  #   ol<internal path>equation": "Chandra-OCR-2",
  #   ne<internal path>equation": "Qwen3.6-VL-REAP",   # B4: only DB-reproducible eq metric (gold CER 0.0506 #1);
  #                                          # Chandra's CDM 0.82 claim is not persisted in runs.
  ```

**Mismatch 2 — `cyrillic` → Qianfan-OCR (soft).**
- *Evidence*: **no usable Cyrillic gold** (the RU gold page CER is 1.8–29 = format mismatch). Ref-free over 57 RU page<internal path>**dots.ocr is the cleanest (rep 0.034)**, olmOCR 0.083, then **Qianfan 0.114**. All three are 100% ok / 0 empty.
- *Verdict*: Qianfan is reliable but **not the cleanest** Cyrillic producer; dots is. The report's robustness matrix shows both at ok%=100. There is **no gold** to break the tie on accuracy, so this is a soft mismatch.
- **Recommendation**: **change to `dots.ocr`** to align the cyrillic override with the global default (one fewer resident model, and the lowest-repetition Cyrillic producer). This is a ref-free pick — label it as such; revisit if/when verified Cyrillic gold arrives. (Keeping Qianfan is also defensible — it was a deliberate dedicated-OCR Cyrillic pick — so present this as a recommendation, not a correction.)

  **Proposed edit** (`_common.py`):
  ```python
  #   ol<internal path>cyrillic": "Qianfan-OCR",
  #   ne<internal path>cyrillic": "dots.ocr",   # B4: ref-free only (no Cyrillic gold). dots cleanest on 57 RU
  #                                   # pages (rep 0.034 < Qianfan 0.114) and == global default.
  ```

### 4b. `engine/config.py`

- `DEFAULT_MODEL = "dots.ocr"` → ✅ **MATCHES.** KEEP. (reliable, perfect tables, prompt-invariant, every language.)
- `DEFAULT_PROMPT = "p00_base"` → ✅ KEEP.
- `ESCALATE_MODEL = "Qwen3.6-VL-REAP"` → ✅ **MATCHES** gold #1 (CER 0.055). KEEP.
- `ROUTING` (per-model best prompts) → ✅ **CONSISTENT with the gold data**, verifie<internal path>- Qwen `form → p01_html_strict`: confirmed (CER 0.000 on p051, 0.114 on 031 — both via p01). ✅
  - Qwen `table → p03_classification_guided`: confirmed (TEDS 0.967 at p03 vs 0.595 at p00). ✅
  - GLM-4.6V `_default → p01_html_strict`, `table → p03`: confirmed (equation p00 loops to CER 2.37; p01 → 0.080; table p03 TEDS 0.625 best for GLM). ✅
  - gemma `p03` catastrophe documente<internal path>gemma-26B form p03 → CER 11.67 (vs p00 0.455) and table p03 in roster notes. ✅ Routing correctly keeps gemma off p03.
  - Non-Latin → p05 for al<internal path>consistent (it is the only prompt run on RU/SV gold). ✅
- **No `engine/config.py` edits recommended.** Its routing table is the validated G1/B2 sweep and is internally consistent with every gold row in the DB.

### Mismatch summary
| Location | Key | Current | Recommended | Type | Gold-backed? |
|---|---|---|---|---|---|
| `_common.py` CHAMPION_MODELS | `equation` | Chandra-OCR-2 | **Qwen3.6-VL-REAP** | change (evidence-led) | yes, n=1 CER |
| `_common.py` CHAMPION_MODELS | `cyrillic` | Qianfan-OCR | **dots.ocr** | change (ref-free) | no — ref-free |
| `_common.py` CHAMPION_MODELS | table/prose_scan/figure | (unchanged) | keep | match | table=gold |
| `engine/config.py` | DEFAULT/ESCALATE/ROUTING | (unchanged) | keep | match | yes |

> Per the hard rules, I did NOT modify any config file — the two edits above are proposals for the orchestrator to apply after review.

---

## 5. Gold coverage / confidence — be honest

**Gold is extremely thi<internal path>6 pages, of which only 3 yield usable CER.**

| Cell | Gold pages | Usable for accuracy? | Why |
|---|---|---|---|
| EN/form | 031_instructions_p001, Capitalism_p051 | ✅ yes (n=2) | HTML gold aligns; CER meaningful. |
| EN/equation | Capitalism_p651 | ✅ yes (**n=1**) | single page — no CI, point estimate only. |
| EN/table | Sahr_Inflation_p009 | ✅ TEDS (**n=1**); CER partial | `score_v3.PENDING` for CER, but TEDS is valid & decisive. |
| SV/prose | SSA_0010 | ❌ no | CER 2–29 for all models = gold/format mismatch (PENDING_VERIFY). |
| RU/cyrillic | NK-SSSR-1923_p031 | ❌ no | CER 1.8–29 for all = gold/format mismatch (PENDING_VERIFY). dense-numeric, user to verify. |

**Bootstrap CIs**: `analyze_battery._bootstrap_ci` requires >1 value per cell to produce a non-degenerate interval. With n=1 (equation, table) the CI collapses to the point estimate — reported honestly as point estimates, NOT as if they were CI-backed. The only multi-page gold cell is EN/form (n=2), still too thin for a meaningful bootstrap. **No champion here rests on a wide, well-sampled gold base.** The picks ar<internal path>table = gold-anchored on structure (TEDS 1.00, decisive even at n=1); equation/form/prose = gold-anchored but n=1–2; cyrillic/swedish = **ref-free only, no gold**.

**Two findings that override report claims (flagged for review):**
1. **Equation champion**: report says Chandra (CDM 0.82); the DB has **no Chandra CER on the equation page** and the only reproducible metric favors Qwen. The CDM metric is not persisted → unverifiable. I recommend Qwen but note the report's claim cannot be checked against the frozen DB.
2. **Cyrillic**: there is **no verified gold at all**; the Qianfan pick is reliability-based, and dots.ocr is actually the cleaner producer. Any Cyrillic accuracy claim awaits the user's dense-numeric gold verification.

**Top caveat**: the entire accuracy axis stands on **3 born-digital/clean-scan English pages**. The cells that matter most for this workspace's hard cases — **Cyrillic dense-numeric tables and historical Swedish scans** — have **zero usable gold**. The matrix is correct about *reliability* everywhere but is only *accuracy-anchored* on clean English. Do not over-trust the n=1 cells; expand gold (especially RU/cyrillic and SV/prose) before freezing those cells as accuracy champions.

---

## 6. Bottom line

The canonical Thunderdome v3 verdict hold<internal path>**default dots.ocr + escalate Qwen3.6-VL-REAP, native llama.cpp**. The frozen matrix refines two routing entrie<internal path>- **equatio<internal path>Chandra-OCR-2 → Qwen3.6-VL-REAP** (evidence-led; the only DB-reproducible equation metric, and already the escalation model).
- **cyrilli<internal path>Qianfan-OCR → dots.ocr** (ref-free; cleanest producer + matches global default).
Everything else (table=dots, prose_scan=dots, figure=describer, DEFAULT/ESCALATE/ROUTING in engine/config.py) is consistent with the gold and stays.
