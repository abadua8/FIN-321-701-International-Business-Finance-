# Appendix — Phase 5 Independent LLM Run (Raw Output)

**Purpose:** Part 1 of the Phase 5 validation exercise. A fresh LLM session (Claude, no conversation history, no other context) was given exactly two documents and asked to compute all hedge outcomes and recommend a strategy — no coaching, no correction mid-run, no workbook results shared.

**Documents provided (verbatim, in full):**
1. `docs/specs/2026-08-07-Badua-pharma-exporter-spec.md`
2. `data/2026-08-07-Badua-market-data.md`

## Prompt sent

> "Attached are two documents for a treasury FX hedging analysis: a technical specification and a market-data memo. Using only these two documents, compute the USD proceeds for all four strategies (no hedge, forward, money-market, and put option) across the specification's sensitivity grid, and produce a call-option outcome if the documents support it. Show your formulas in named-range notation. Conclude with a recommended hedge strategy for the CFO. Do not ask me for the underlying workbook — work from these two documents only."

## Raw output (as returned by the fresh session)

---

**FX Hedge Analysis — pharma-exporter scenario**

**Step 1 — Derived inputs (live, per market-data memo)**
- `DF_USD = 1 + R_USD × T_DAYS / BASIS_USD = 1 + 0.0401 × 365/360 = 1.040657`
- `DF_FC = 1 + R_FC × T_DAYS / BASIS_FC = 1 + 0.02884 × 365/365 = 1.028840`
- `F0_in = S0_in × DF_USD / DF_FC = 1.1525 × 1.040657 / 1.028840 = 1.165737`
- `FV_PREM_PUT = −PREM_PUT × FC_AMT × DF_USD = −0.021 × 8,000,000 × 1.040657 = −$174,830.37`

**Step 2 — Forward hedge**
`USD_FWD = FC_AMT × F0_in = 8,000,000 × 1.165737 = $9,325,898`

**Step 3 — Money-market hedge**
1. Borrow EUR: `FC_AMT / DF_FC = 8,000,000 / 1.028840 = €7,775,747`
2. Convert at spot: `€7,775,747 × 1.1525 = $8,961,549`
3. Invest to maturity: `$8,961,549 × 1.040657 = $9,325,898`
Parity check: ties exactly to the forward, as expected under CIP.

**Step 4 — Option (put) hedge**
Premium today: `PREM_PUT × FC_AMT = 0.021 × 8,000,000 = $168,000`.
`USD_PUT(S_T) = MAX(S_T, K_PUT) × FC_AMT + FV_PREM_PUT`

**Step 5 — Sensitivity grid**
Per §6 of the spec, the grid is `S_T = S0_in × (1 + n × STEP_FRAC)` for `n = −5…+5`, `STEP_FRAC = 1%`, spanning §6's stated range **1.0346 → 1.1435** around the spec's base case (`S0_in = 1.0890`):

| n | S_T | USD_NO_HEDGE | USD_FWD | USD_MM | USD_PUT |
|---|---|---:|---:|---:|---:|
| −5 | 1.03455 | $8,276,400 | $9,325,898 | $9,325,898 | $9,045,170 |
| −4 | 1.04544 | $8,363,520 | $9,325,898 | $9,325,898 | $9,045,170 |
| −3 | 1.05633 | $8,450,640 | $9,325,898 | $9,325,898 | $9,045,170 |
| −2 | 1.06722 | $8,537,760 | $9,325,898 | $9,325,898 | $9,045,170 |
| −1 | 1.07811 | $8,624,880 | $9,325,898 | $9,325,898 | $9,045,170 |
| 0 | 1.08900 | $8,712,000 | $9,325,898 | $9,325,898 | $9,045,170 |
| +1 | 1.09989 | $8,799,120 | $9,325,898 | $9,325,898 | $9,045,170 |
| +2 | 1.11078 | $8,886,240 | $9,325,898 | $9,325,898 | $9,045,170 |
| +3 | 1.12167 | $8,973,360 | $9,325,898 | $9,325,898 | $9,045,170 |
| +4 | 1.13256 | $9,060,480 | $9,325,898 | $9,325,898 | $9,045,170 |
| +5 | 1.14345 | $9,147,600 | $9,325,898 | $9,325,898 | $9,045,170 |

Across this grid, the forward and money-market hedges dominate every row, and the put option never rises above its floor because every grid point sits at or below `K_PUT = 1.1525`.

**Call option** — Not computed. §2.1 of the spec marks `K_CALL` / `PREM_CALL` as "N/A — not used" for this receivable-only scenario, and while the market-data memo lists a live `K_CALL` value, it does not give a premium-derivation method or a sourced quote for `PREM_CALL` ("derived from `PREM_PUT` via put-call parity" is stated, but no formula is given). Rather than invent a parity convention, I'm leaving the call outcome unquantified — flag this if a call comparison is required.

**Recommendation:** Across the full sensitivity grid, the forward hedge (equivalently, the money-market hedge under parity) produces the highest USD proceeds at every tested point, with zero variance. The put option provides no upside in this range — every S_T is at or below the strike — while still costing $174,830 in present-value terms. Given no house view on EUR direction has been formed and the forward dominates the entire tested range, **I recommend the forward hedge** (or the money-market hedge if forward-line capacity is constrained) for the full EUR 8,000,000 notional. The put would only be preferable if EUR is expected to appreciate beyond roughly 1.1525 — outside the range this sensitivity table covers.

---

**End of raw LLM output.**
