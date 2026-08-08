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
