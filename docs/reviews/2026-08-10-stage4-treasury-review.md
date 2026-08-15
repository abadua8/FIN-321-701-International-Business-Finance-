# Stage 4 review — pharma exporter market data & population · Treasury sign-off

Allan Jay — one line in your inputs table is the best modelling decision anyone made this stage:

> "`PREM_CALL` is not an independent input — it is derived from `PREM_PUT` via put-call parity in `Inputs!C20` and updates automatically."

Everyone else carried two independent premium assumptions. That is quietly incoherent: with a forward at a premium to spot, an at-the-money put and an at-the-money call *cannot* cost the same, and hardcoding both leaves the model asserting an arbitrage. You made the call premium a *consequence* of the put premium and the forward, so the two can never drift apart. That is the difference between a model that holds numbers and a model that holds relationships.

| Criterion | Score |
|---|---|
| Data quality & provenance | 50 / 50 |
| Model resolves cleanly | 33 / 33 |
| Lab cross-check | 17 / 17 |
| **Total** | **100 / 100** |

**What else you did well**

- **You timestamped to the second and recorded the quote's own quality.** `S0_in` 1.1525 at "2026-08-07, 23:01:54 GMT+1 (delayed quote)," with the day's range 1.1523–1.1526 and previous close 1.1555. Flagging the feed as *delayed* and giving the range lets a reader judge whether your single number is representative — a mid taken from a tight range is worth more than one taken from a volatile session.
- **You explained why the Euribor source is a mirror, not the primary.** "via euribor-rates.eu (24h-delayed public mirror of the official EMMI fixing)... EMMI's own site requires paid/registered access for same-day data." Naming the authoritative source, the reason you could not use it, and what you used instead is exactly how a proxy gets defended.
- **You justified `R_USD` against the alternative you rejected.** 1-Yr Treasury CMT "not SOFR" — arguing against the competing choice is stronger than asserting your own.
- **You documented the forward sources that failed.** Barchart, Investing.com, and FXEmpire all behind paid feeds. That turns a CIP-implied forward from a shortcut into a documented fallback.
- **`K_CALL` is labelled as not used in the decision.** "At-the-money placeholder, per scenario convention; not used in the receivable decision." A EUR call does nothing for a EUR receivable, and saying so stops a reader treating the column as a hedge candidate.

**One precise correction — your two day-count bases**

Your CIP formula uses different denominators per leg:

```
DF_USD = 1 + R_USD × T_DAYS/360
DF_FC  = 1 + R_FC  × T_DAYS/365
```

Splitting the basis by currency rather than applying one convention to both is more rigorous than most of the cohort managed, and the instinct is right — day-count is a property of the instrument, not of the model.

But check the specific instrument. **12-month Euribor quotes on ACT/360, not ACT/365.** The 365 basis belongs to sterling money markets (and to euro-area *government bond* yields), not to Euribor. So your EUR leg should also be `/360`.

It is worth real money. Your `R_FC` is 2.884% over 365 days: on ACT/365 that accrues 2.884%, on ACT/360 it accrues 2.924% — about 4bp more. Pushed through CIP, your forward moves roughly 0.00046, or about **$3,700 on EUR 8,000,000**. Small, but it is a convention error rather than a rounding one, and it will scale with the notional and the differential.

The fix is one cell. If you want to keep the two bases as separate named ranges — which is the right structure — set them both to 360 and document *why* each is 360, so the next person inherits the reasoning rather than a coincidence.

**To push it further**

- **Your parity residual is still the tautology you identified at Stage 3.** You spotted it there and left a note for this stage; the note was correct, and `F0_in` is still CIP-derived because no live quote was reachable. Say so explicitly in this memo too — a reader arriving at Phase 4 without your Stage 3 audit will read the passing check as market validation.
- **Rename the workbook.** It is still `2026-08-07-Badua-{scenario-slug}-model (2).xlsx` — literal braces, plus a duplicate copy.

**Next — Stage 5**

Hand the workbook and Stage 2 spec to an LLM, get its analysis, then break it. Recompute at least three outputs by hand with the arithmetic written out — forward proceeds, the put floor, and the crossover spot where the put overtakes the forward. Write the recommendation in a CFO's voice framed on risk tolerance. Your spec retrospective has strong material ready: the `#N/A` annotation bug, the $322 hand-rounding gap, and the tautological parity check you called out before anyone asked.

— Treasury

---

### How to work this review — professional workflow

Treat this PR the way an analyst treats feedback from Treasury — a review is a proposal to engage with, not a checklist to rubber-stamp:

1. **Read it yourself first.** Understand each point and form your own view before changing anything. Disagreeing *with a documented reason* is a legitimate, senior response.
2. **Stress-test it with an LLM (pushback pass).** Paste this review and your spec into your AI assistant and ask it to (a) explain anything you're unsure of more deeply, and (b) argue the *other side* — where might the reviewer be wrong, and what would you give up by making each change. You're building judgment, not just executing edits.
3. **Decide, then draft the changes with the LLM.** For the points you accept, have the AI help implement them — you specify exactly what and why. Your spec is the prompt; precise in, correct out.
4. **Verify — non-negotiable.** Re-run your own checks (`scripts/recalc.py`, the parity tie-out, sensitivity continuity, no error cells) and confirm the numbers before you commit. An AI will hand you a confident wrong edit; verification is what makes the result *yours*.
5. **Close the loop on the PR.** Reply in the thread with what you changed, what you pushed back on and why, then commit and push. Writing down the reasoning is exactly how this works on a real team.

*This is the same human-in-the-loop discipline the whole project is built on: the LLM drafts, you edit and verify, and you own the result.*
