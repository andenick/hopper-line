# Hopper Line v2 — The Method (HL2)

**What this is**: the distilled method specification of the Hopper Line v2 local extraction engine,
public-safe. The full internal protocol document (with infrastructure specifics) is not shipped;
everything needed to understand and rebuild the method is here and in
[ROUTING.md](ROUTING.md). Companion evidence: [../results/](../results/).

## 0. Scope and boundaries

- **HL2 is a local-GPU engine**: offline, zero-API, one consumer GPU (RTX 5090 class, ~8–24 GB
  VRAM budget), open-weight models served via llama.cpp (gguf) and native runtimes.
- It is **not** the agent-reading protocol family (HDARP): different mechanism (model routing, not
  prompt/validation discipline over a frontier model), different failure taxonomy, different cost
  profile. The two are complementary; results from one are never claimed for the other.
- Outputs: the four content artifacts per document — body text, tables (CSV/HTML), equations
  (LaTeX), figures/charts (structured descriptions + data) — with per-artifact provenance
  (model, method, score).

## 1. Governing principles

1. **Research → Reproduce → Race → Gate → Freeze.** No model or threshold becomes a default
   without a CI-gated win on **real** gold. Synthetic evals are stress-tests only — a lesson
   learned when the synthetic champion (dots.ocr-default) was overturned by real-gold racing
   (GLM-OCR won every structural task by large margins).
2. **Born-digital bypass first.** Never OCR a page with a usable text layer.
3. **Per-region routing (the R9 rule).** Route each *region*, not each page: specialists and
   doc-models for STRUCTURAL extraction (tables, scanned text, equations, layout); strong general
   VLMs for REASONING (charts). R9 is the spine of the engine.
4. **Cheap-default + confidence-gated escalation.** The cheap model reads first; only flagged
   regions escalate to the expensive one (e.g. dots.ocr → dots.mocr on abstention; empty-output
   gates → olmOCR-2-7B).
5. **Everything is flagged and provenanced.** Estimated data (chart series) is never primary;
   low-confidence regions go to a review queue; every artifact carries its model + method + score.

## 2. Pipeline stages

```
PDF → text-layer bypass check
    → layout detection (DocLayout-YOLO) + reading order (XY-Cut / XY-Cut++)
    → per-region dispatch by content type (ROUTING.md)
    → extraction by the routed model (cheap default)
    → CPU verification (cross-foot table checks, schema validation, confidence gating)
    → escalation on gate failure (the expensive specialist)
    → artifact assembly + provenance + review queue
```

Serving discipline (one GPU): a single model server with slot parallelism (`-np 4`, greedy +
seeded, KV-cache q8), **restart every ~100 pages** (server aging measured ~52% slowdown per 100
requests), diff-gate canaries between restarts to prove output stability (0 violations observed).
Measured throughput: ~20–25 pages/min sustained, ~30k pages/day.

## 3. The routing table

The definitive, evidence-cited table — content × era → model → config → verifier → escalation —
is [ROUTING.md](ROUTING.md). Every cell carries its gold-set citation. Highlights:

- Tables: GLM-OCR with long-context config (`-c 32768`, ≥24k output tokens) for dense archival
  tables — a ship-pair discovered by racing (engine cap fix + config).
- TOC/leader-dot "tables" are **rerouted to body text** by a CPU triage (precision = recall = 1.0
  on the labeled cluster) — the table models were wasted there.
- Charts: REAP held a frozen tie against a challenger (won scanned-synthetic, tied real) —
  incumbency under evidence-parity, recorded as a user-gated switch, not a silent flip.

## 4. Quality gates (what "no silent quality loss" means mechanically)

- **Cross-model confidence gating**: model-disagreement predicts error (Spearman −0.98 vs CER on
  degraded pages) — disagreement flags route to review, not to the output silently.
- **Cross-foot verification** on tables (column/row sums reconcile) with cell-recall reporting;
  low-recall pages flagged.
- **Schema validation** for figure descriptions (completeness + grammar-of-output), with a
  "null over guess" prompt policy — an honest empty beats an invented description.
- **Bake-off regression gate**: a config change ships only if every stratum's score holds within
  noise of its recorded champion (the final bake-off: 17 strata, zero regressions).
- **Refuted-candidate ledger**: losing candidates and their numbers are recorded next to winners
  (see ROUTING.md notes and FROZEN_CHAMPIONS.md) — the negative results are part of the method.

## 5. Extensions built on the engine

- **Filename renamer (V2)**: a 3-model compose consensus (lead + cross-check + tertiary composer,
  different lineages/architectures) converting title-page reads into `[YYYY] Author - Title`
  filenames, with an empirically calibrated confidence formula
  (`0.30·year_agree + 0.30·author_agree + 0.40·mean_title_jaccard`). See FROZEN_CHAMPIONS.md.
- **Escalation tiers**: the mode-C ladder (cheap → gated specialist) generalizes to any new
  content type with its own race.

## 6. What you need to rebuild it

Open-weight models named in ROUTING.md (dots.ocr / dots.mocr, GLM-OCR, Chandra-OCR-2, Qwen3-VL-8B,
Qwen3.6-VL-REAP, Uni-MuMER, olmOCR-2-7B, DocLayout-YOLO, Nemotron-Parse), an llama.cpp-compatible
server per model, and the verification utilities implied by §4 (cross-foot checker, schema
validator, confidence gating). The gold sets are named per stratum in the results docs; assembling
equivalents from the named public sources is straightforward (see METRICS.md for the lists).

---

*This method description is part of the Hopper Line evidence repository. License: MIT.*
