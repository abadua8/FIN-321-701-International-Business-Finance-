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
