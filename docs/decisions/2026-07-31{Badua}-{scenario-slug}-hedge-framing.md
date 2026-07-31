Hedging the EUR 4,500,000 Receivable Due in One Year

Created by: [Allan Jay Badua] Updated by: [Allan Jay Badua] Date Created: [July 31, 2026] Date Updated: July 31, 2026 Version: 1.0 LLM Used: Claude

Executive Summary

Our firm expects a EUR 4,500,000 payment in one year. Because the euro amount is fixed but the dollar amount is not, an ordinary currency swing could shift USD proceeds by roughly $450,000 — a real budget risk, not a tail event. This memo frames that exposure in dollar terms and lays out three hedging approaches — a forward, a money-market hedge, and a put option — with honest trade-offs between certainty and upside. Rather than recommend one immediately, we propose a five-stage process: specify a hedging model, build and audit it, populate it with live market data, and validate the result before committing capital. We are requesting approval to begin Stage 2.

Background & Objectives

The firm has a EUR 4,500,000 receivable settling in one year. Because we hold the currency risk until settlement, our USD cash flow depends on where EURUSD lands on that date rather than today. At EURUSD 1.10, proceeds are $4,950,000; at 1.00, proceeds are $4,500,000 — a $450,000 gap between two entirely plausible outcomes, given that a 9% move is well within a normal year for this pair. Left unhedged, this exposure flows directly into our USD revenue forecast, cash planning, and covenant headroom.

The primary objective is to protect the dollar value of this receivable against adverse EUR moves. A secondary objective is to do so without giving up more upside than necessary if the euro strengthens in our favor, and without committing to a hedge before we have priced the alternatives against each other on a consistent basis.

Methods

We will evaluate three hedge families, each representing a different trade-off between certainty and upside:

Family	Mechanics	Pro	Con
Forward / futures	Contract today to sell EUR at a locked rate on the settlement date	Complete certainty — budget is safe	Give up all upside if EUR strengthens; counterparty/margin mechanics
Money-market hedge	Borrow EUR today, convert at spot, invest the USD — a synthetic forward built from interest rates	Same certainty as a forward; useful when forwards are unavailable or mispriced	Uses borrowing capacity; more moving parts to execute and unwind
Options (put floor)	Buy the right — not the obligation — to sell EUR at a strike	Floor under proceeds, plus upside participation if EUR rallies	Premium paid up front, whether or not the hedge is ever used

In plain terms: a forward is selling the house forward at a fixed price. An option is fire insurance — you pay a premium and hope to waste it.

To choose among them on a rigorous basis, we will build a model in five stages:

Model specification — design the workbook on paper: named ranges, tabs, formula logic, validation checks.
AI-assisted build + audit — generate the workbook from the spec, then audit and correct the output line by line.
Live market data — replace placeholders with real, sourced, timestamped forward points, interest rates, and option premiums.
Validation & recommendation — run an independent check, verify by hand, and deliver a final recommendation with sensitivity across a range of ending spot rates.

Ahead of the formal build, we are cross-checking early numbers against the FX Hedging Lab, a live workbook that computes the forward, the money-market hedge, and the option payoff together, and that will serve as our check-figure oracle once our own model is built.

Limitations & Next Steps

This analysis assumes the EUR 4,500,000 amount and one-year timing are firm; if either changes, the hedge size and tenor must be revisited. It also depends on forward points, interest rates, and option premiums that are not yet live — the figures above are illustrative until Stage 4. We have not yet formed a house view on the direction of EUR, so no family is being recommended at this stage; that judgment belongs in the Stage 5 recommendation, once real pricing is in hand.

Immediate next steps:

Treasury / FX Risk: build the Stage 2 model specification and circulate for review.
CFO: approve proceeding to Stage 2 on this timeline, or flag a preferred hedge family to prioritize in the build.
References
FX Hedging Lab. (2026). FX Hedging Lab: Forward, Money-Market, and Option Hedge Calculator. Retrieved July 31, 2026, from https://adamwstauffer.github.io/ai-lms/fxlab.html
