# Phase 5 Validation — pharma-exporter Hedge Model

**Scenario:** U.S. Pharmaceutical Exporter, EUR 8,000,000 receivable due in 365 days
**Author:** Allan Jay Badua
**Date:** 2026-08-14
**Workbook validated:** `models/builds/2026-08-07-Badua-pharma-exporter-model.xlsx`, version 1.1 (live data)
**Inputs to the independent LLM run:** `docs/specs/2026-08-07-Badua-pharma-exporter-spec.md` and `data/2026-08-07-Badua-market-data.md` only
**Raw LLM output:** [`2026-08-14-Badua-pharma-exporter-validation-appendix.md`](./2026-08-14-Badua-pharma-exporter-validation-appendix.md)

---

## Part 1 — Independent LLM Execution

A fresh Claude session, with no prior conversation and no access to the workbook, was given only the Phase 2 spec and the Phase 4 market-data memo and asked to compute all hedge outcomes and recommend a strategy. The exact prompt and full raw output are in the linked appendix. No coaching or mid-run correction was given.

**Headline result:** the fresh session correctly derived every formula, correctly substituted live values from the market-data memo for `S0_in`, `R_USD`, `R_FC`, and reproduced the forward, money-market, and put-floor figures exactly. It went wrong in exactly one place: **it built the sensitivity grid around the spec's illustrative example values (`S0_in = 1.0890`, the grid literally printed in spec §6: "1.0346, 1.0454, ... 1.1435") instead of rebuilding the grid around the live `S0_in = 1.1525`** that the same memo had just given it two paragraphs earlier. It also declined to compute a call-option outcome, citing the spec's "N/A" language and the absence of a stated premium formula.

## Part 2 — Comparison & Hand Verification

### 2a. Comparison table — LLM result vs. workbook, three S_T points

| Strategy | S_T point | LLM (fresh session) | Workbook (live, v1.1) | Δ | Diagnosis |
|---|---|---:|---:|---:|---|
| No Hedge | n=−5 | $8,276,400 | $8,759,000 | −$482,600 | **LLM error.** Used stale `S_T = 1.089×0.95 = 1.03455` instead of live `S_T = 1.1525×0.95 = 1.094875`. |
| No Hedge | n=0 (baseline) | $8,712,000 | $9,220,000 | −$508,000 | **LLM error**, same root cause — even the *baseline* row used the old placeholder spot instead of the live one. |
| No Hedge | n=+5 | $9,147,600 | $9,681,000 | −$533,400 | **LLM error**, same root cause. |
| Forward | all 3 points | $9,325,898 (flat) | $9,325,898 (flat) | $0 | Match. Forward proceeds don't depend on `S_T`, so the grid error is invisible here. |
| Money Market | all 3 points | $9,325,898 (flat) | $9,325,898 (flat) | $0 | Match, same reason — and confirms parity holds in both runs. |
| Put | n=−5 | $9,045,170 | $9,045,170 | $0 | **Coincidental match.** Both the stale and live `S_T` are below `K_PUT = 1.1525` at this row, so the floor binds either way and the grid error happens not to matter here. |
| Put | n=0 | $9,045,170 | $9,045,170 | $0 | Match — baseline sits exactly at the strike in both runs. |
| Put | n=+5 | $9,045,170 | $9,506,170 | −$461,000 | **LLM error, most consequential instance.** The LLM's stale `S_T` (1.14345) never exceeds the *live* `K_PUT` (1.1525), so its put table shows the floor holding at every single row — the put's entire upside-participation story silently disappears. |
| Call | any point | Not computed | $8,939,272 (baseline) / $9,506,170 (ceiling) | N/A | **Spec ambiguity, not an error.** The spec marks the call family "N/A," the memo populates a live `K_CALL` without a premium method, and neither document states the put-call-parity formula the workbook actually uses (`PREM_CALL = PREM_PUT + (F0_in − K_CALL)/DF_USD`). The LLM's refusal to invent a premium was the more defensible choice given what it was handed. |

**Bottom line:** three of five strategies (Forward, Money Market, and the downside leg of the Put) are grid-invariant or coincidentally unaffected, so the LLM's numbers there are correct. The two places the error actually surfaces — No-Hedge proceeds and the Put's upside participation — are exactly the two things a CFO needs to see correctly to judge the trade-off between certainty and optionality. The LLM's recommendation ("forward dominates the entire tested range") is a direct, logical consequence of its narrowed, stale grid — it never tested a `S_T` above 1.1525, so it never saw the region where No Hedge or the Put overtake the Forward.

### 2b. Hand-verification table (calculator + named-range notation, no Excel)

**1. Forward proceeds — `USD_FWD = FC_AMT × F0_in`**

```
DF_USD = 1 + R_USD × T_DAYS / BASIS_USD
       = 1 + 0.0401 × (365/360)
       = 1 + 0.0401 × 1.013889
       = 1 + 0.040657
       = 1.040657

DF_FC  = 1 + R_FC × T_DAYS / BASIS_FC
       = 1 + 0.02884 × (365/365)
       = 1 + 0.02884
       = 1.028840

F0_in  = S0_in × DF_USD / DF_FC
       = 1.1525 × 1.040657 / 1.028840
       = 1.199357 / 1.028840
       = 1.165737   (matches workbook Inputs!C15 to 6 dp)

USD_FWD = FC_AMT × F0_in
        = 8,000,000 × 1.165737
        = $9,325,896          [hand, 6-dp F0_in]
   vs. workbook: $9,325,898.13 [full float precision]
   Δ ≈ $2 — pure rounding noise from truncating F0_in to 6 decimal places by hand; not a real discrepancy.
```

**2. Money-market hedge — all three legs**

```
Step 1 — Borrow EUR today:
  EUR_BORROWED = FC_AMT / DF_FC = 8,000,000 / 1.028840 = €7,775,747
  Check: 7,775,747 × 1.028840 ≈ 8,000,000 ✓

Step 2 — Convert to USD at spot:
  USD_CONVERTED = EUR_BORROWED × S0_in
                = 7,775,747 × 1.1525
                = 7,775,747 + (7,775,747 × 0.15) + (7,775,747 × 0.0025)
                = 7,775,747 + 1,166,362 + 19,439
                = $8,961,548     (workbook: $8,961,548.93 ✓)

Step 3 — Invest to maturity:
  USD_MM = USD_CONVERTED × DF_USD
         = 8,961,548 × 1.040657
         = 8,961,548 + (8,961,548 × 0.040657)
         = 8,961,548 + 364,349
         = $9,325,897     (workbook: $9,325,898.13 — Δ ≈ $1, rounding)

Parity check: USD_MM − USD_FWD ≈ $0 ✓ (tautological by construction, since
F0_in is itself CIP-derived from the same R_USD/R_FC/S0_in — documented
in the build audit, Finding 3, and still true after Phase 4 since no
live outright forward quote was ever sourced.)
```

**3. Option (put) outcome — baseline, `S_T = S0_in = K_PUT`**

```
Premium paid today = PREM_PUT × FC_AMT = 0.021 × 8,000,000 = $168,000

FV_PREM_PUT = −PREM_PUT × FC_AMT × DF_USD
            = −168,000 × 1.040657
            = −(168,000 + 168,000×0.040657)
            = −(168,000 + 6,830.36)
            = −$174,830.36     (workbook: −$174,830.37 ✓)

USD_PUT(baseline) = MAX(S_T, K_PUT) × FC_AMT + FV_PREM_PUT
                   = 1.1525 × 8,000,000 − 174,830.36
                   = 9,220,000 − 174,830.36
                   = $9,045,169.64    (workbook: $9,045,169.63 ✓, Δ < $0.01)
```

All three hand-computed figures reconcile to the workbook within a few dollars — the only source of divergence is decimal truncation in the by-hand intermediate steps, not a modeling error on either side.

---

## Part 4 — Spec Retrospective

**What the LLM got wrong, and why:** the fresh session's only real error — the stale sensitivity grid — traces directly to a spec authoring choice, not to a gap in financial reasoning. Spec §6 states the grid *rule* correctly (`S_T = S0_in × (1 ± 5%)` in 1% steps) but then, in the same breath, prints the fully-expanded example grid for the *original placeholder scenario* (`1.0346, 1.0454, ... 1.1435`, built off `S0_in = 1.0890`). To a model reading linearly, a concrete list of eleven numbers is a stronger signal than an abstract formula sitting one sentence earlier — and the market-data memo, which supplies the new live `S0_in = 1.1525`, never restates the full grid, only four summary point-values (`USD_FWD`, `USD_MM`, `USD_FLOOR_PUT`, `USD_CEILING_CALL`). Those four happen to be exactly the values that are either grid-invariant (`USD_FWD`, `USD_MM`) or landed at the floor by coincidence at the specific points the memo chose to report — so nothing in the memo forced the LLM to notice its grid was stale. The spec's own worked example became an anchor precisely because it looked authoritative and the live-data memo gave no reason to distrust it.

**What this reveals about the spec:** a document written to be "precise enough that an AI... could build the complete workbook from this document alone" (spec's own framing line) still mixed a general formula with a scenario-specific worked example inside the same section, and never marked the worked numbers as disposable. The second gap — the call-option premium — is smaller but real: the spec explicitly declines to specify a parity formula because the call "isn't used," but Phase 5's own comparison requirement (this document) needs a call figure anyway, so the spec under-serves a downstream reader it didn't anticipate.

**What v2 would say differently:**
1. Any worked example tied to placeholder inputs gets a visible expiry marker — e.g. "*Example only, computed at the Phase-2 placeholder `S0_in = 1.0890`; recompute this entire grid from the formula above using whatever `S0_in` is live at build time.*" — rather than trusting a reader to infer that a specific number is disposable.
2. §2.1's `K_CALL` / `PREM_CALL` row should state the put-call-parity formula explicitly (`PREM_CALL = PREM_PUT + (F0_in − K_CALL)/DF_USD`) even while marking the call "not used in the receivable decision" — completeness of the named-range contract shouldn't depend on whether a downstream stage turns out to need it.
3. The market-data memo's practice of reporting four summary values instead of a full recomputed table is efficient for a human but insufficient as a re-grounding checkpoint for an LLM reader — Phase 4 memos in future scenarios should include the full recomputed `S_T_grid`, or at minimum flag explicitly that any illustrative grid printed in the spec is now stale.

"The spec was perfect" is not the finding here — the spec was precise on formulas and genuinely ambiguous on which of its own numbers were live versus illustrative, and a fresh reader followed that ambiguity exactly where it led.
