# Hedge Recommendation — EUR 8,000,000 Receivable (pharma-exporter)

**To:** Chief Financial Officer
**From:** Allan Jay Badua, Treasury Analyst / FP&A
**Date:** 2026-08-14
**Re:** Recommended FX hedge for the EUR 8,000,000 receivable due in 365 days
**Supporting model:** `models/builds/2026-08-07-Badua-pharma-exporter-model.xlsx` (v1.1, live market data as of 2026-08-07); validated in `analysis/2026-08-14-Badua-pharma-exporter-validation.md`

---

## A. Exposure Summary

Our European customers owe us EUR 8,000,000, settling in 365 days. The euro amount is fixed by contract; the dollar amount we actually collect is not, because it depends on EURUSD on the settlement date. At today's live spot (1.1525), that receivable is worth $9,220,000. A routine one-year move in this currency pair — well within the ±5% to ±9% range this pair has covered historically — can shift realized USD proceeds by $450,000 to over $850,000 in either direction. Left unhedged, that swing flows straight into our USD revenue forecast, cash planning, and covenant headroom.

## B. Hedge Outcomes

Four strategies were priced on a common basis using live market data (spot, U.S. Treasury 1-Yr CMT, 12M Euribor) retrieved 2026-08-07:

| Strategy | Baseline proceeds (S_T = spot) | Behavior across the range | Upfront cost |
|---|---:|---|---:|
| **No hedge** | $9,220,000 | Moves 1-for-1 with spot; $8,759,000–$9,681,000 across a ±5% move | $0 |
| **Forward** | $9,325,898 (locked, invariant) | Flat at $9,325,898 regardless of where spot lands | $0 (rate is embedded) |
| **Money market** | $9,325,898 (locked, invariant) | Identical to the forward — a synthetic replication, confirmed by an exact parity tie in the model | Uses borrowing capacity instead of a forward line |
| **Put option (K = 1.1525, ATM)** | $9,045,169 floor | Floors at $9,045,169 if EUR weakens; rises 1-for-1 with spot, minus the premium, if EUR strengthens | $168,000 today ($174,830 future value) |

The forward and money-market hedges are economically identical here — the model's parity check ties them to the penny — so the choice between them is really about which credit facility (forward line vs. borrowing capacity) the treasury desk prefers to use, not about the payoff itself.

## C. Sensitivity Interpretation

The four strategies trade off certainty, flexibility, and cost in a specific, quantifiable way:

- **If EUR weakens (S_T below spot):** No hedge is the worst outcome by construction — proceeds fall as low as $8,759,000 at a 5% depreciation. The forward and money-market hedges are the best protection available, locking in $9,325,898 regardless of how far EUR falls. The put floors losses at $9,045,169 — better than no hedge, but $280,728 below the forward's locked-in level, because an at-the-money put only protects from today's spot, while the forward captures the full USD–EUR interest-rate carry (the forward rate, 1.165737, sits about 115 basis points above spot).
- **If EUR strengthens (S_T above spot):** No hedge and the put both participate in the upside; the forward and money-market hedges do not — they stay flat at $9,325,898 no matter how far EUR rallies. No hedge overtakes the forward once EUR appreciates roughly 1.15% above today's spot (i.e., past the forward rate itself). The put overtakes the forward at a slightly higher bar — roughly 3.0% above spot — because it first has to earn back its premium.
- **The trade-off in one sentence:** the forward and money-market hedges buy total certainty at the cost of all upside; the put buys a worse (but still solid) floor plus unlimited upside, for a known, bounded premium; no hedge buys full upside at the cost of full downside exposure.

## D. Recommendation

**Recommended strategy: the put option, on the full EUR 8,000,000 notional.**

The spec for this project defined our objective explicitly: *protect the USD value of the receivable against EUR depreciation while retaining upside if EUR strengthens.* Of the four strategies priced, the put is the only one that satisfies both halves of that mandate at once — the forward and money-market hedges satisfy only the protection half (at zero premium cost, but by fully surrendering upside), and no hedge satisfies only the upside half. Since no house view on EUR direction has been formed, giving up an entire side of the distribution — as the forward does — is a stronger bet on direction than the mandate calls for.

**If the CFO's priority is instead pure budget certainty** (e.g., the receivable is already earmarked against a fixed USD obligation, and any variance, favorable or not, is unwelcome), the forward or money-market hedge is the better fit, and either is acceptable — the choice between them should be driven by which facility (forward line vs. borrowing capacity) treasury prefers to use, not by economics, since the two are priced identically under parity.

**A middle path** — hedging roughly half the notional via forward/money-market and buying a put on the remainder — is a reasonable compromise if the $174,830 premium is a budget concern but full upside surrender is not acceptable either; this halves both the premium outlay and the upside given up, at the cost of a more complex position to administer.

## E. Executive Justification

- **Cash-flow stability:** the put guarantees a minimum of $9,045,169 regardless of where EUR lands, closing off the worst-case scenarios that matter most for cash planning.
- **Budget certainty:** slightly weaker than the forward's guarantee ($9,045,169 vs. $9,325,898 floor), but the gap is a known, bounded, quantifiable cost ($174,830) — not an open-ended risk.
- **Liquidity:** the premium is paid once, upfront, in cash ($168,000) — no ongoing margin calls or collateral posting, unlike an exchange-traded alternative, and it is small relative to the $9.2M receivable it protects (about 1.8%).
- **Optionality:** full participation in EUR appreciation is preserved past the ~3% breakeven versus the forward — directly relevant given the firm has not formed a directional view and does not want to foreclose the upside case.
- **Premium cost:** the $174,830 future-value cost is the explicit, bounded price of carrying both protection and optionality simultaneously; it should be weighed against the value of certainty (forward/MM, cost = all upside) and the value of doing nothing (cost = full downside exposure, an average $461,000–$533,000 swing at a 5% move against us in this analysis).

I'm available to walk through the sensitivity model directly, or to price the middle-path (partial hedge) structure in more detail, ahead of a final decision.
