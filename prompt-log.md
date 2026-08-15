7/24:
  create a 150–200 word BIO.md about me, Allan Jay Badua, an incoming senior student at Shidler College of Business majoring in MIS and Finance. I will also be studying abroad at Yonsei university in South Korea for Fall 2026
    rewrite to first person

7/24:format this resume into a resume.md file for github. fix any errors you may come across

8/7: # Prompt Log — `pharma-exporter` Stage 3 Spec

Scenario: U.S. Pharmaceutical Exporter, EUR 8,000,000 receivable due in 365 days, S0_in = 1.0890.
Deliverable: `docs/specs/2026-08-07-Badua-pharma-exporter-spec.md`

---

## Entry 1 — Initial draft

**Prompt to Claude:**
> "Fill out `template-spec.md` for a U.S. pharmaceutical exporter with a EUR 8,000,000 receivable due in one year, spot EURUSD 1.0890. Use the standardized named ranges (FC_AMT, S0_in, F0_in, R_USD, R_FC, K_PUT, K_CALL, PREM_PUT, PREM_CALL, T_DAYS), flag every market input as indicative, and work the calculation flow through in named-range notation."

**What Claude produced:** A first pass of §§1–8 with the named-range table, tab architecture, and calculation flow for forward, money-market, and put strategies, using placeholder rates (R_USD 5.25%, R_FC 3.00%) and a derived forward rate.

---

## Entry 2 — Iteration example (gap found → fix)

**Gap found:** In the first draft, Claude computed `F0_in` directly as a flat placeholder (it picked a round number, 1.10, without deriving it from `R_USD`, `R_FC`, and `S0_in`) and then, separately, ran the money-market hedge in §5 using the covered-interest-parity formula. The two numbers didn't tie out — the parity check in Step 3 showed a >2% gap between `USD_FWD` and `USD_MM`, which is disqualifying under the Auditability Checklist ("Money-market hedge ties to forward hedge within 0.05%"). The gap was an internal-consistency error, not a real parity violation: Claude had treated `F0_in` as an independent free input instead of deriving it from the same rate inputs used in the money-market walk.

**Fix applied (human edit):** Rewrote §5 Step 1 to explicitly derive `F0_in = S0_in × DF_USD / DF_FC` from the already-declared `R_USD`, `R_FC`, `T_DAYS` inputs (1.0890 × 1.0532 / 1.0300 ≈ 1.1136), and annotated it in §2.1 as "(indicative, derived via covered interest parity — see §4, Step 1)" rather than an independently chosen number. Recomputed `USD_FWD` and `USD_MM` off the corrected `F0_in`; the parity check now closes to within 0.01% ($8,908,800 vs. $8,908,553), which is the behavior the Phase-3 audit checklist expects. Also added an explicit note that at Phase 4, the live dealer forward quote replaces this derived figure and the two should track closely but need not match exactly — so a future analyst doesn't mistake the derivation for the real market forward.

**Why this matters:** The whole point of the named-range contract is that an AI or colleague can rebuild the workbook from the spec with no independent judgment calls. A forward rate that isn't tied back to the same interest-rate inputs driving the money-market leg would fail the model's own parity check the moment someone built it in Excel — exactly the kind of error the spec is supposed to prevent before any cell exists.

---

## Entry 3 — Secondary check

**Prompt to Claude:**
> "Recompute the base-case table in §7.1 using the corrected F0_in, and confirm the put payoff at S_T = K_PUT matches K_PUT × FC_AMT + FV_PREM_PUT exactly."

**Result:** Base-case table regenerated ($8,712,000 / $8,908,800 / $8,908,553 / $8,535,057 for no hedge / forward / money market / option); kink check confirmed exact match at the strike. No further edits needed.

# Prompt Log — `pharma-exporter` Stage 3 Spec

Scenario: U.S. Pharmaceutical Exporter, EUR 8,000,000 receivable due in 365 days, S0_in = 1.0890.
Deliverable: `docs/specs/2026-08-07-Badua-pharma-exporter-spec.md`

---

## Entry 1 — Initial draft

**Prompt to Claude:**
> "Fill out `template-spec.md` for a U.S. pharmaceutical exporter with a EUR 8,000,000 receivable due in one year, spot EURUSD 1.0890. Use the standardized named ranges (FC_AMT, S0_in, F0_in, R_USD, R_FC, K_PUT, K_CALL, PREM_PUT, PREM_CALL, T_DAYS), flag every market input as indicative, and work the calculation flow through in named-range notation."

**What Claude produced:** A first pass of §§1–8 with the named-range table, tab architecture, and calculation flow for forward, money-market, and put strategies, using placeholder rates (R_USD 5.25%, R_FC 3.00%) and a derived forward rate.

---

## Entry 2 — Iteration example (gap found → fix)

**Gap found:** In the first draft, Claude computed `F0_in` directly as a flat placeholder (it picked a round number, 1.10, without deriving it from `R_USD`, `R_FC`, and `S0_in`) and then, separately, ran the money-market hedge in §5 using the covered-interest-parity formula. The two numbers didn't tie out — the parity check in Step 3 showed a >2% gap between `USD_FWD` and `USD_MM`, which is disqualifying under the Auditability Checklist ("Money-market hedge ties to forward hedge within 0.05%"). The gap was an internal-consistency error, not a real parity violation: Claude had treated `F0_in` as an independent free input instead of deriving it from the same rate inputs used in the money-market walk.

**Fix applied (human edit):** Rewrote §5 Step 1 to explicitly derive `F0_in = S0_in × DF_USD / DF_FC` from the already-declared `R_USD`, `R_FC`, `T_DAYS` inputs (1.0890 × 1.0532 / 1.0300 ≈ 1.1136), and annotated it in §2.1 as "(indicative, derived via covered interest parity — see §4, Step 1)" rather than an independently chosen number. Recomputed `USD_FWD` and `USD_MM` off the corrected `F0_in`; the parity check now closes to within 0.01% ($8,908,800 vs. $8,908,553), which is the behavior the Phase-3 audit checklist expects. Also added an explicit note that at Phase 4, the live dealer forward quote replaces this derived figure and the two should track closely but need not match exactly — so a future analyst doesn't mistake the derivation for the real market forward.

**Why this matters:** The whole point of the named-range contract is that an AI or colleague can rebuild the workbook from the spec with no independent judgment calls. A forward rate that isn't tied back to the same interest-rate inputs driving the money-market leg would fail the model's own parity check the moment someone built it in Excel — exactly the kind of error the spec is supposed to prevent before any cell exists.

---

## Entry 3 — Secondary check

**Prompt to Claude:**
> "Recompute the base-case table in §7.1 using the corrected F0_in, and confirm the put payoff at S_T = K_PUT matches K_PUT × FC_AMT + FV_PREM_PUT exactly."

**Result:** Base-case table regenerated ($8,712,000 / $8,908,800 / $8,908,553 / $8,535,057 for no hedge / forward / money market / option); kink check confirmed exact match at the strike. No further edits needed.

---

## Entry 4 — Phase 3 build request

**Prompt to Claude:**
> "Here is my model specification. Build the workbook it specifies. Every requirement in the spec's build contract is binding: all ten named ranges, formulas only (no pasted values), cover page, Legend/Key tab with the color convention, all three hedge families, the ±5% sensitivity table with chart, and the validation checks computed in the workbook. Give me the result as a downloadable .xlsx file. If anything in the spec is ambiguous, ask me questions before you build." [spec pasted in full]

**Ambiguity flagged before building:** The spec marks `K_CALL`/`PREM_CALL` as "N/A — not used" (receivable-only scenario), but the assignment's build contract requires all ten named ranges attached to cells *and* both a put and a call hedge family with proceeds as a function of `S_T`. Asked the user how to reconcile this rather than silently picking one reading.

**Answer:** Add indicative placeholders and a parallel Options_Call tab that mirrors the put logic, so all ten named ranges are attached and the put-and-call requirement is satisfied. `K_CALL` defaulted to `S0_in` (at-the-money, matching `K_PUT`'s convention); `PREM_CALL` derived via put-call parity (`PREM_CALL = PREM_PUT + (F0_in − K_CALL) / DF_USD`) rather than picked arbitrarily, so the placeholder is at least internally consistent.

---

## Entry 5 — Build execution and self-audit

**What Claude did:** Built the workbook with `openpyxl` (Cover, Legend-Key, Inputs, Forward, MoneyMarket, Options_Put, Options_Call, Sensitivity, Notes-Validation — 9 tabs), attached all 22 named ranges (10 required + 12 supporting), wrote every calculated cell as a formula referencing named ranges, added a formula-driven ±5%/1%-step sensitivity grid with a line chart, and wrote in-workbook validation checks (parity, put-kink, grid-symmetry) as live formulas.

**First `recalc.py` run:** `status: errors_found`, 6× `#N/A` in `Inputs!F13:F21`. Root cause: descriptive text notes had been written starting with `"= ..."`, which Excel/LibreOffice parses as a formula rather than text. Fixed by rewording those six cells to start with `"Note: ..."`; second `recalc.py` run returned `status: success`, `total_errors: 0`, `total_formulas: 145`.

**Independent verification (not just recalc):** Pulled every named range's target cell via `wb.defined_names` and every check-figure's computed value via a `data_only=True` load, and hand-verified a sample of Sensitivity-grid rows against the underlying formulas (e.g. put payoff at `S_T = 1.09989` → `1.09989 × 8,000,000 − 176,942.5 = 8,622,177.5`, matches). Findings logged in `analysis/2026-08-07-Badua-build-audit.md`.

---

## Entry 6 — Phase 4 live-data population

**Prompt to Claude:**
> "Replace the placeholder inputs (S0_in, R_USD, R_FC, K_PUT, K_CALL) with live, sourced, timestamped market data. PREM_PUT stays a scenario assumption. Document every source and retrieval timestamp in a market-data memo, recalculate the workbook, and re-run every validation check."

**What Claude did:** Retrieved live spot (Yahoo Finance EURUSD=X), U.S. Treasury 1-Yr CMT (R_USD), and 12M Euribor via EMMI/euribor-rates.eu (R_FC); entered all five values into the existing `Inputs` tab (no structural changes — the named-range contract held without a single formula edit); re-ran `recalc.py` (0 errors, 145 formulas); re-verified parity, put-kink, and grid-symmetry checks (all PASS); and additionally cross-checked the live-data forward/money-market outputs against the FX Hedging Lab (adamwstauffer.github.io/ai-lms/fxlab.html), diagnosing and documenting a reconciled $3,629 (0.039%) gap traced to a day-count-basis convention difference (this workbook splits ACT/360-USD/ACT/365-EUR; the lab uses T/360 uniformly) — both tools internally correct, no workbook change made. Full sourcing, rationale, and the cross-check writeup are in `data/2026-08-07-Badua-market-data.md`.

---

## Entry 7 — Phase 5 independent validation (fresh-session test of the spec + memo)

**Prompt to Claude (fresh session, no history, spec + market-data memo only, verbatim):**
> "Attached are two documents for a treasury FX hedging analysis: a technical specification and a market-data memo. Using only these two documents, compute the USD proceeds for all four strategies (no hedge, forward, money-market, and put option) across the specification's sensitivity grid, and produce a call-option outcome if the documents support it. Show your formulas in named-range notation. Conclude with a recommended hedge strategy for the CFO. Do not ask me for the underlying workbook — work from these two documents only."

**What the fresh session got right:** every formula (DF_USD, DF_FC, F0_in, FV_PREM_PUT), correct substitution of the live market-data-memo values, exact-to-the-dollar forward, money-market, and put-floor figures.

**What it got wrong:** built the sensitivity grid around the spec's illustrative placeholder example (`S0_in = 1.0890`, the literal grid printed in spec §6) instead of rebuilding it around the live `S0_in = 1.1525` the memo supplied. This silently erased the put option's upside participation across the entire tested range and produced a "forward always wins" recommendation that doesn't hold once the correct live grid is used — the put and no-hedge strategies both overtake the forward once EUR appreciates far enough, and the correct grid reaches into that territory while the LLM's stale one never does. It also correctly declined to compute a call-option premium, since neither document states the put-call-parity formula the workbook actually uses — a legitimate call-out, not an error.

**Fix applied:** none to the LLM's output — per the exercise's design, the run was not corrected. The discrepancy, its root cause, and its downstream effect on the recommendation are documented and diagnosed in full in `analysis/2026-08-14-Badua-pharma-exporter-validation.md` (Parts 1–2), with the raw output preserved verbatim in the linked appendix, and traced back to a specific spec-authoring gap in the Part 4 retrospective (mixing a general grid formula with a scenario-specific worked example in the same section, with no marker distinguishing the two).

**Why this matters:** this is exactly the kind of error hand-auditing catches and a recalc pass never would — every number the fresh LLM produced was internally consistent and formula-correct; the workbook it implied would show zero formula errors. The mistake was upstream, in which inputs it chose to trust, not in how it computed with them.
