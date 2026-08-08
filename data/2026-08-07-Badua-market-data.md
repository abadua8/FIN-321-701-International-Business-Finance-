# Market-Data Memo — Phase 4 Live Data Population

**Scenario:** pharma-exporter · EUR 8,000,000 receivable due in 365 days
**Author:** Allan Jay Badua
**Data retrieved:** 2026-08-07 (all timestamps below are as-quoted by the source; retrieval performed same-day)
**Workbook:** `2026-08-07-Badua-pharma-exporter-model.xlsx`, version 1.1

This memo documents every input now live in the workbook's `Inputs` tab, where it came from, and how to re-pull it. Two inputs (`FC_AMT`, `T_DAYS`) are contractual and were not re-sourced; `PREM_PUT` was kept as a scenario assumption per the phase instructions.

## Inputs table

| Named Range | Live Value | Unit | Source | Retrieval Timestamp | Proxy / Computation |
|---|---|---|---|---|---|
| `S0_in` | 1.1525 | USD/EUR | Yahoo Finance, `EURUSD=X` (finance.yahoo.com/quote/EURUSD=X) | 2026-08-07, 23:01:54 GMT+1 (delayed quote) | Direct spot mid; day's range 1.1523–1.1526, previous close 1.1555 |
| `R_USD` | 4.01% | Annual %, 1-Yr CMT | U.S. Department of the Treasury, Daily Treasury Par Yield Curve Rates (home.treasury.gov) | 2026-08-07 (official close, published same day) | None — direct read of the "1 Yr" column, 08/07/2026 row |
| `R_FC` | 2.884% | Annual %, 12M Euribor | EMMI (European Money Markets Institute), via euribor-rates.eu (24h-delayed public mirror of the official EMMI fixing) | 2026-08-06 fixing (most recent published at time of retrieval; EMMI's own site requires paid/registered access for same-day data) | None — direct 12M Euribor fixing |
| `F0_in` | 1.165737 | USD/EUR | Computed (CIP), formula already in `Inputs!C15` | 2026-08-07 (derived from the three inputs above) | `F0_in = S0_in × DF_USD / DF_FC` where `DF_USD = 1 + R_USD×T_DAYS/360`, `DF_FC = 1 + R_FC×T_DAYS/365`. No live outright 1Y EUR/USD forward quote was accessible through free sources (Barchart, Investing.com, and FXEmpire forward-rate pages all required a paid/registered feed at time of retrieval) — see comparison below. |
| `K_PUT` | 1.1525 | USD/EUR | Set to live `S0_in` | 2026-08-07 | At-the-money, per scenario strike convention (put struck at spot) |
| `K_CALL` | 1.1525 | USD/EUR | Set to live `S0_in` | 2026-08-07 | At-the-money placeholder, per scenario convention; not used in the receivable decision (see `Options_Call` tab note) |
| `PREM_PUT` | 0.0210 | USD per 1 EUR | Scenario-given (unchanged) | — | **Assumption**, not re-sourced: no live OTC option-premium feed was available. Kept exactly as issued in the original scenario, per phase instructions. `PREM_CALL` is not an independent input — it is derived from `PREM_PUT` via put-call parity in `Inputs!C20` and updates automatically. |
| `FC_AMT` | 8,000,000 | EUR | Scenario contract (unchanged) | — | Firm; sales contract / AR aging schedule |
| `T_DAYS` | 365 | Days | Scenario contract (unchanged) | — | Firm; contract settlement date minus build date |

## Rate-choice rationale

**R_USD — 1-Yr Treasury Constant Maturity (DGS1 / Treasury par yield curve), not SOFR.** The build-contract source note originally pointed to 12M Term SOFR. I substituted the Treasury 1-Yr CMT because it is (a) published daily by the U.S. Treasury at a fixed, well-defined tenor with no interpolation or compounding-convention ambiguity, (b) freely and permanently accessible without a data-vendor login, and (c) a standard proxy for the risk-free USD rate over a 1-year horizon in academic CIP exercises. Term SOFR is arguably the "more correct" market rate for a bank's actual funding cost, but it sits behind CME/ICE licensing for live values; the Treasury CMT gets economically very close (both are short-term risk-free USD proxies) and is fully auditable. **The choice of proxy matters more than the basis-point gap between the two** — a reader who disagrees can swap in Term SOFR without changing the model's structure.

**R_FC — 12-Month Euribor, not €STR.** The original source note referenced 12M €STR (a swap curve). €STR itself is an overnight rate; a "12M €STR" figure is really a 1-year OIS swap rate, which again sits behind a data-vendor paywall for live values. 12-Month Euribor is the standard, freely published EUR-area interbank reference at the matching 1-year tenor, is the direct EUR analogue to R_USD's role in the CIP formula, and is what most textbook and classroom CIP exercises use. It is not a perfect frictionless risk-free rate (it embeds bank credit risk), but neither is R_USD's CMT proxy a pure risk-free rate in practice — the two are consistent proxies for their respective interbank funding costs.

## Forward comparison: live CIP-implied vs. original indicative

| | Spot | R_USD | R_FC | F0 (CIP) | F0 − S0 |
|---|---|---|---|---|---|
| **Original scenario placeholder** | 1.0890 | 5.25% | 3.00% | 1.11356 | +0.0246 (+2.25%) |
| **Live, 2026-08-07** | 1.1525 | 4.01% | 2.884% | 1.16574 | +0.0132 (+1.15%) |

The live forward premium (EUR trading forward at a premium to USD spot) is smaller than the placeholder implied — about half the size in percentage terms. That's a direct consequence of the USD–EUR rate differential narrowing: the placeholder assumed a 225 bp gap (5.25% − 3.00%); live data shows roughly a 113 bp gap (4.01% − 2.884%). Since forward points are driven almost entirely by that differential under CIP, a narrower gap compresses the forward premium proportionally. The euro also appreciated materially against the dollar between the scenario's build date and the live-data retrieval date (spot moved from 1.0890 to 1.1525, +5.8%), which shifts the whole forward curve up in absolute USD/EUR terms even though the *proportional* forward premium shrank.

No live tradeable outright forward quote could be pulled from a free source at the time of retrieval, so this comparison is CIP-implied-vs-CIP-implied rather than CIP-implied-vs-market-quoted. If a live dealer forward becomes available (e.g., via a Bloomberg terminal), the gap between that quote and the CIP-implied `F0_in` above would isolate the cross-currency basis — the residual not explained by the pure interest-rate differential.

## Workbook population — what changed structurally

**Nothing.** All ten named-range core inputs plus `K_PUT`/`K_CALL` were entered directly into their existing `Inputs` tab cells; every downstream formula (`DF_USD`, `DF_FC`, `F0_in`, `FV_PREM_PUT`, `PREM_CALL`, `FV_PREM_CALL`, and every tab that consumes them — `Forward`, `MoneyMarket`, `Options_Put`, `Options_Call`, `Sensitivity`) recalculated without modification. This is expected and by design: the named-range contract in `Inputs` is the only tab meant to change hands-on, and it held. No formula had to be fixed.

## Post-population validation (all PASS after recalculation)

| Check | Result | Where |
|---|---|---|
| Parity: `USD_MM` ties to `USD_FWD` within 0.05% | **PASS** (0.00% — exact tie, as expected under CIP since `F0_in` is itself CIP-derived) | `MoneyMarket!C16` |
| Put kink check at `S_T = K_PUT` | **PASS** | `Options_Put!C13` |
| Sensitivity grid symmetric around `S0_in` | **PASS** | `Sensitivity!C18` |
| Zero formula errors workbook-wide | **PASS** (145 formulas, 0 errors via LibreOffice recalculation) | — |

Recomputed headline figures at the new live spot (1.1525):
- `USD_FWD` (forward-hedge proceeds): **$9,325,898**
- `USD_MM` (money-market-hedge proceeds): **$9,325,898** (parity holds exactly, since `F0_in` is CIP-derived from the same `R_USD`/`R_FC`/`S0_in` used in the money-market replication)
- `USD_FLOOR_PUT` (worst case across the ±5% grid): **$9,045,170**
- `USD_CEILING_CALL` (best case across the ±5% grid, placeholder tab): **$9,506,170** at the top of the grid — not the recommendation driver for this receivable-only scenario

## FX Hedging Lab cross-check — RESOLVED

Entered the live inputs (`FC_AMT = 8,000,000`; `S0_in = 1.1525`; `R_USD = 4.00%`; `R_FC = 3.68%` in the lab's own demo scenario, then re-run with our actual live `R_FC = 2.884%`; `T_DAYS = 365`; `K_PUT = 1.1525`; `PREM_PUT = 0.021`) into the lab at https://adamwstauffer.github.io/ai-lms/fxlab.html and compared its Forward hedge / Money-market hedge / interest-rate-parity outputs against this workbook.

**Root cause of the discrepancy, confirmed:** exactly candidate #1 from the original open item — a day-count basis mismatch, not an error in either tool.

- **This workbook** splits the day-count basis by currency, per market convention: `DF_USD = 1 + R_USD × T/360` (ACT/360, standard for USD money markets) and `DF_FC = 1 + R_FC × T/365` (ACT/365, standard for EUR money markets). This is documented as an explicit assumption in `Notes-Validation!A15`.
- **The FX Hedging Lab** applies `T/360` to *both* legs uniformly (visible in its own formula labels: `FC_AMT / (1 + R_FC·T/360)` for the borrow step, and `S0 × (1+R_USD·T/360)/(1+R_FC·T/360)` for the implied forward) — a simplified single-basis convention common in introductory treatments.

Recomputing our live inputs under the lab's uniform-T/360 convention:

| Quantity | This workbook (ACT/360 USD, ACT/365 EUR) | Lab convention (T/360 both legs) | Delta |
|---|---|---|---|
| `DF_USD` | 1.040657 | 1.040657 | — (USD leg identical in both; both use T/360) |
| `DF_FC` | 1.028840 | 1.029241 | +0.000401 |
| `F0_in` (implied forward) | 1.165737 | 1.165284 | −0.000454 USD/EUR (≈4.5 pips) |
| `USD_FWD` | $9,325,898.13 | $9,322,268.71 | −$3,629.41 (−0.039%) |
| `USD_MM` | $9,325,898.13 | $9,322,268.71 | −$3,629.41 (identical to `USD_FWD` in both — parity holds exactly *within* each tool's own convention) |
| `FV_PREM_PUT`, baseline `USD_PUT` | $9,045,169.63 | $9,045,169.63 | $0 (put payoff only touches the USD leg's `DF_USD`, which both conventions compute identically) |

**Resolution:** Both tools are internally correct — each ties its own money-market hedge to its own forward hedge exactly (MM − Forward = $0.00 in both), because each derives its implied forward from the *same* `R_USD`/`R_FC`/`S0_in` triple via CIP, just with a different EUR-leg day-count basis. The 0.039% gap between the two tools' dollar outputs is exactly the "which basis convention" choice — it sits comfortably inside the workbook's own 0.05% parity-tolerance band, so it wouldn't itself flag a FAIL, but it explains why a side-by-side dollar comparison shows a real, non-error difference of about $3,629 on a $9.3M notional. The workbook's ACT/360-USD / ACT/365-EUR split was kept as the more market-accurate convention (matching actual USD and EUR money-market practice); the lab's uniform T/360 is a reasonable simplification for an introductory tool. No change was made to the workbook as a result of this cross-check — the two are reconciled, not in conflict.

The lab also separates a "quoted forward" (manual input) from an "implied forward" (its own CIP calculation) and checks the two against each other — the same comparison this workbook makes conceptually between `F0_in` (CIP-derived, since no live outright quote was found) and the original scenario's indicative placeholder forward, documented above.

## Reproducibility notes

- Yahoo Finance quotes are "delayed" (per the page's own disclosure) and update continuously; re-pulling minutes later will show a slightly different number. The timestamp above is what was displayed at retrieval.
- The Treasury par yield curve is an *official, end-of-day* series — re-pulling on the same date will return the identical 4.01% figure; it only changes the next business day.
- The Euribor figure from euribor-rates.eu is a 24-hour-delayed mirror of the EMMI fixing; the *official* real-time source (emmi-benchmarks.eu) requires registration. Re-pulling same-day may show the prior day's fixing until EMMI publishes the current one at ~11:00 CET.
