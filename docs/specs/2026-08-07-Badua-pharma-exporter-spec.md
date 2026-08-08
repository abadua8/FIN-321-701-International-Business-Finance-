<div style="border-top: 6px solid #024731; border-bottom: 1px solid #B2B2B2; padding: 12px 0; margin-bottom: 24px; font-family: 'Open Sans', Helvetica, Arial, sans-serif;">
  <div style="color: #024731; font-weight: 700; letter-spacing: 0.06em; text-transform: uppercase; font-size: 0.85rem;">University of Hawaiʻi at Mānoa · Shidler College of Business</div>
  <div style="color: #000000; font-weight: 700; font-size: 1.25rem; margin-top: 4px;">FIN-321 International Finance &amp; Securities</div>
  <div style="color: #525252; font-weight: 400; font-size: 0.95rem;">FX Transaction Hedging Project — Technical Specification</div>
</div>

# U.S. Pharmaceutical Exporter — FX Transaction Hedge Model · Technical Specification

> <span style="color:#024731; font-weight:700;">Technical specification</span> for the FX transaction hedge model — the named-range contract, calculation flow, and validation checks, precise enough that an AI or a colleague who never saw the framing memo could build the complete workbook from this document alone. This spec is the Phase 3 build prompt.

| Field | Value |
|------|------|
| **Created by** | Allan Jay Badua |
| **Updated by** | Allan Jay Badua |
| **Date Created** | 2026-08-07 |
| **Date Updated** | 2026-08-07 |
| **Version** | 0.1 |
| **LLM Used** | Claude — drafted from this spec's inputs and the Stage 2 framing memo, then edited by hand (see `prompt-log.md`) |
| **Role** | Treasury Analyst / FP&A Analyst |
| **Audience** | CFO / Director of Treasury |
| **Scenario slug** | `pharma-exporter` |
| **Companion Workbook** | `docs/spreadsheets/International Finance Spreadsheets.xlsx` (Chapter 8 Transaction Hedging tabs — reference/worked example) or student build |

---

## 1. Problem Statement

The firm, a U.S. Pharmaceutical Exporter, holds a EUR 8,000,000 receivable from European customers settling in 365 days. Because the euro leg is fixed but the dollar leg floats with spot EURUSD, an ordinary one-year currency swing (±5–9%) can move realized USD proceeds by roughly $400,000–$800,000, flowing directly into USD revenue forecasting, cash planning, and covenant headroom. This specification documents the analytical framework used to quantify and compare four strategies — **no hedge**, **forward hedge**, **money-market hedge**, and **option (put) hedge** — and to produce the sensitivity evidence that supports the Stage 4 hedging recommendation. All market inputs below are **indicative — replaced with live market data at Phase 4.**

- **Exposure type / functional currency:** Receivable, EUR-denominated; functional currency USD.
- **Amount / quote convention / settlement:** EUR 8,000,000; quoted USD per EUR; settlement in 365 days from spec date.
- **Objective:** Protect the USD value of the receivable against EUR depreciation while retaining upside if EUR strengthens, and price all three hedge families on one consistent basis before committing capital.
- **Decision context:** Corporate treasury recommendation to the CFO; no house view on EUR direction has been formed, so no family is being recommended ahead of Phase 4 live pricing.

---

## 2. Inputs (Named-Range Contract)

### 2.1 Core Inputs

| Named Range | Description | Placeholder Value | Unit | Phase-4 Data Source |
|---|---|---:|---|---|
| `FC_AMT` | Foreign-currency notional (receivable) | 8,000,000 | EUR | Sales contract / AR aging schedule |
| `S0_in` | Spot exchange rate at inception | 1.0890 | USD per EUR | Bloomberg BGN spot mid, timestamped at model build |
| `F0_in` | Forward rate to settlement (365 days) | 1.1136 *(indicative, derived via covered interest parity — see §4, Step 1)* | USD per EUR | Bank forward quote or Bloomberg forward points (BGN FWD), timestamped |
| `R_USD` | USD interest rate to settlement | 5.25% | Annual %, ACT/360 | 12M Term SOFR / USD swap curve |
| `R_FC` | EUR interest rate to settlement | 3.00% | Annual %, ACT/365 | 12M €STR swap curve |
| `T_DAYS` | Days to settlement | 365 | Days | Contract settlement date − spec date |
| `BASIS_USD` | USD-leg day-count denominator | 360 | Days | Market convention (ACT/360) |
| `BASIS_FC` | EUR-leg day-count denominator | 365 | Days | Market convention (ACT/365) |
| `K_PUT` | Put option strike | 1.0890 *(defaulted to `S0_in`, at-the-money)* | USD per EUR | Desk quote at strike selection |
| `PREM_PUT` | Put premium, USD per 1 EUR | 0.0210 | USD | OTC dealer quote / Bloomberg OVML |
| `K_CALL` | Call option strike | N/A | USD per EUR | Not used — this scenario is receivable-only. Retained as a named range for tab-architecture symmetry with the payable variant (§4, Step 7). |
| `PREM_CALL` | Call premium, USD per 1 EUR | N/A | USD | Not used — see `K_CALL` note. |

### 2.2 Derived / Intermediate Values

| Name | Description | Formula (named-range notation) | Indicative Value |
|---|---|---|---:|
| `DF_USD` | USD accumulation factor to settlement | `1 + R_USD × T_DAYS / BASIS_USD` | 1.0532 |
| `DF_FC` | EUR accumulation factor to settlement | `1 + R_FC × T_DAYS / BASIS_FC` | 1.0300 |
| `FV_PREM_PUT` | Future value of put premium at settlement | `−PREM_PUT × FC_AMT × DF_USD` | −$176,943 |
| `S_T_grid` | Sensitivity spot grid at settlement | `S0_in × (1 + n × STEP_FRAC)`, `n = −5…+5`, `STEP_FRAC = 1%` | 1.0346 → 1.1435 |
| `USD_NO_HEDGE` | USD proceeds under no hedge | `S_T × FC_AMT` | varies by row |

---

## 3. Tab Architecture

| Tab | Purpose |
|---|---|
| **Cover** | Title, scenario name (`pharma-exporter`), preparer, dates, version, one-paragraph problem statement. |
| **Legend / Key** | Cell-color legend (yellow = input, blue = assumption, black = formula, green = cross-tab link, dark red = external link) and named-range index. |
| **Inputs** | All `§2.1` named ranges with units, Phase-4 source, and access-timestamp column. The only tab an analyst edits for scenario work. |
| **Forward** | Step 2 calculation: `USD_FWD = FC_AMT × F0_in`. |
| **Money-Market** | Step 3 three-leg walk (borrow → convert → invest) plus the parity check against `USD_FWD`. |
| **Options (Put)** | Step 4 premium and payoff schedule: `USD_PUT(S_T)` across `S_T_grid`. |
| **Sensitivity** | Step 5 grid: `USD_NO_HEDGE`, `USD_FWD`, `USD_MM`, `USD_PUT` and hedge-profit columns across all 11 `S_T_grid` rows, plus the winner-label columns and the comparison chart. |
| **Notes & Assumptions** | §3 assumptions, data sources with access dates, and the Appendix A change log. |

---

## 4. Assumptions & Constraints

- **Quote convention:** All rates in USD per EUR; a higher quote means EUR appreciation.
- **Horizon:** Single-maturity model, `T_DAYS = 365`.
- **Day-count basis:** `DF_USD = 1 + R_USD × T_DAYS / BASIS_USD` (ACT/360); `DF_FC = 1 + R_FC × T_DAYS / BASIS_FC` (ACT/365). Split bases are used from the outset (see Model Review, §6.2 of the class template) rather than a single shared `BASIS`.
- **Parity:** The money-market hedge is assumed to replicate the forward hedge under covered interest-rate parity; any residual gap is a test of parity in the quoted inputs, not a model error.
- **Option premium:** Paid upfront in USD, quoted per 1 unit of EUR, no contract multiplier. Treated as a negative cash flow at t₀ and carried forward at `R_USD` (`FV_PREM_PUT`) to put it on the same footing as settlement-date proceeds.
- **Counterparty / credit risk:** Excluded; all derivatives assumed frictionless and creditworthy.
- **Transaction costs / bid-ask spreads:** Excluded from the base case; flagged as a sensitivity candidate in §6.
- **Tax / accounting treatment:** Excluded. Pre-tax cash outcomes only.
- **Scenario construction:** `S_T` is varied deterministically across the grid; no probability weights or implied-volatility distribution applied.

---

## 5. Calculation Flow

Written in named-range notation, receivable exposure. All values indicative pending Phase-4 live data.

**Step 1 — Derived inputs**
1. `DF_USD = 1 + R_USD × T_DAYS / BASIS_USD` → 1.0532
2. `DF_FC = 1 + R_FC × T_DAYS / BASIS_FC` → 1.0300
3. `FV_PREM_PUT = −PREM_PUT × FC_AMT × DF_USD` → −$176,943
4. *(Indicative only, for internal consistency)* `F0_in = S0_in × DF_USD / DF_FC` → 1.1136 — Phase 4 replaces this derived figure with a live dealer forward quote; the two should track closely but need not match exactly.

**Step 2 — Forward hedge**
- `USD_FWD = FC_AMT × F0_in` → 8,000,000 × 1.1136 ≈ **$8,908,800**, locked at t₀, invariant across `S_T_grid`.

**Step 3 — Money-market hedge (3 steps + parity check)**
1. Borrow `FC_AMT / DF_FC` EUR today → 8,000,000 / 1.0300 ≈ €7,766,990.
2. Convert to USD at spot: `(FC_AMT / DF_FC) × S0_in` ≈ €7,766,990 × 1.0890 ≈ **$8,458,333**.
3. Invest to maturity: `USD_MM = (FC_AMT / DF_FC) × S0_in × DF_USD` ≈ $8,458,333 × 1.0532 ≈ **$8,908,553**.
   - **Parity check:** `USD_MM ≈ USD_FWD` (8,908,553 vs. 8,908,800 — within rounding). A persistent gap at Phase 4 flags a covered-interest-parity violation in the live inputs, not a model error.

**Step 4 — Option hedge (put floor with upside)**
- Pay `PREM_PUT × FC_AMT` = $168,000 today for a put on EUR at `K_PUT = 1.0890`.
- For each `S_T` in `S_T_grid`: `USD_PUT(S_T) = MAX(S_T, K_PUT) × FC_AMT + FV_PREM_PUT`.
- At `S_T = S0_in = K_PUT` (baseline): `USD_PUT = 1.0890 × 8,000,000 − 176,943 = 8,712,000 − 176,943 ≈` **$8,535,057**.
- `USD_FLOOR_PUT = MIN(USD_PUT)` across the grid = the value at any `S_T ≤ K_PUT`, i.e. **$8,535,057** (since `K_PUT = S0_in`, the floor and baseline coincide in this scenario).

**Step 5 — Sensitivity table (see §6 for the grid and §7 for validation)**

**Step 6 — Summary metrics**
- `USD_FLOOR_PUT` = $8,535,057 (worst-case put outcome on the grid).
- `USD_BASE_k` = `USD_k` evaluated at `S_T = S0_in` for each strategy — feeds the §7 base-case table.

**Step 7 — Payable variant (not used this scenario)**
- Not applicable: this is a receivable-only build. `K_CALL` / `PREM_CALL` are retained as named ranges for architectural symmetry with a future payable tab (buy forward, borrow-USD/invest-FC money market, call option), per §2.1.

---

## 6. Sensitivity Plan

- **Grid:** `S_T_grid` spans `S0_in × (1 ± 5%)` in 1% increments → 11 rows including the baseline (`S_T = S0_in = 1.0890`): 1.0346, 1.0454, 1.0563, 1.0672, 1.0781, **1.0890**, 1.0999, 1.1108, 1.1217, 1.1326, 1.1435.
- **Strategies plotted:** no hedge, forward, money market, option (put).
- **Primary chart:** one line chart, `S_T` on the x-axis, USD proceeds on the y-axis. No-hedge is linear through the origin; forward and money-market are horizontal (locked-in); option is piecewise-linear with a kink at `K_PUT = S0_in`.
- **Secondary table:** hedge profit vs. no hedge (`USD_k − USD_NO_HEDGE`) for each strategy, per row, to make the visual trade-off numeric.
- **What the chart communicates:** certainty (forward / money-market, flat) vs. optionality (put, kinked) vs. naked exposure (no hedge, unbounded both directions).

---

## 7. Validation Rules (Check Figures / Phase 3 Audit Checklist)

- [ ] `USD_MM` ties to `USD_FWD` within 0.05% (parity check; §5, Step 3).
- [ ] Put payoff at `S_T = K_PUT` equals `K_PUT × FC_AMT + FV_PREM_PUT` (kink verification) → confirms at $8,535,057.
- [ ] `USD_PUT(S_T)` is continuous (no jump) across the strike — payoff transitions smoothly from the flat floor segment to the linear upside segment.
- [ ] No error cells (`#REF!`, `#VALUE!`, `#DIV/0!`) anywhere in the workbook.
- [ ] Every output cell is a formula referencing named ranges — no hard-coded results, no bare `$A$1`-style cell references.
- [ ] `S_T_grid` is symmetric around `S0_in` and driven by a single `STEP_FRAC` input (1%), not hard-coded per row.
- [ ] Notes tab records the access date/timestamp for every live-market input at Phase 4.

### 7.1 Base-Case Values (`S_T = S0_in`)

| Strategy | USD Proceeds (Receivable) | Hedge Profit vs. No Hedge |
|---|---:|---:|
| No hedge | $8,712,000 | — |
| Forward | $8,908,800 | +$196,800 |
| Money market | $8,908,553 | +$196,553 |
| Option (put) | $8,535,057 | −$176,943 |

*(All figures indicative; recompute once Phase-4 live inputs replace the placeholders in §2.1.)*

---

## 8. Outputs

| Output | Description | Format |
|---|---|---|
| `INPUT_PANEL` | All §2.1 named-range inputs with units, sources, access dates | Top of Inputs tab |
| `STRATEGY_SUMMARY` | `USD_FWD`, `USD_MM`, `USD_BASE_k` per strategy, plus `USD_FLOOR_PUT` | Table above the sensitivity grid |
| `SENSITIVITY_TABLE` | USD proceeds for each strategy across `S_T_grid` (±5%, 1% steps) | Table on Sensitivity tab |
| `HEDGE_PROFIT_TABLE` | `USD_k − USD_NO_HEDGE` per strategy, per row | Sub-table beside `SENSITIVITY_TABLE` |
| `WINNER_LABELS` | `ARGMAX` labels (overall winner incl. no hedge; best active hedge excl. no hedge) | Two label columns |
| `SENSITIVITY_CHART` | Line chart, USD outcome vs. `S_T`, all four strategies | Embedded chart, Sensitivity tab |
| `EXEC_SUMMARY` | 1–2 paragraph narrative with explicit recommendation | Stage 4 memo (downstream deliverable) |

---

## 9. Limitations & Next Steps

**Limitations.** This specification does not incorporate: partial / layered / dynamic hedging (static, full-notional hedge at t₀ only); credit, counterparty, or settlement risk; implied-volatility-based option pricing (`PREM_PUT` is a scenario input, not a Black-Scholes output); hedge-accounting treatment (ASC 815 / IFRS 9); or multi-currency / multi-horizon portfolio effects. `F0_in`, `R_USD`, `R_FC`, and `PREM_PUT` are all indicative and will be replaced with sourced, timestamped live-market figures at Phase 4.

**Next steps.** Phase 3 builds the workbook from this spec (named ranges, tabs, and calculation flow as written above), then audits it line by line against §7. Phase 4 replaces every indicative input with live, sourced data and re-runs §5–§7. Phase 5 validates the rebuilt model and delivers the CFO recommendation memo.

---

## Appendix A — Change Log

| Version | Date | Author | Change |
|---|---|---|---|
| 0.1 | 2026-08-07 | Allan Jay Badua | Initial draft, populated from `template-spec.md` using the `pharma-exporter` scenario parameters (EUR 8,000,000 receivable, S0_in 1.0890). Drafted with Claude, then hand-edited — see `prompt-log.md` for the iteration record. |
