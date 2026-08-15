# Stage 5 review — pharma exporter LLM analysis & validation · Treasury sign-off

Allan Jay — the diagnosis in your retrospective is the best piece of writing in this cohort's Stage 5 set, and it is worth naming precisely why. You did not stop at "the LLM used the wrong spot." You traced it to a specific authoring decision in your own spec: §6 states the grid *rule* correctly, then prints the fully-expanded eleven-number grid computed off the Phase-2 placeholder `S0_in = 1.0890` — and, as you put it, *"to a model reading linearly, a concrete list of eleven numbers is a stronger signal than an abstract formula sitting one sentence earlier."* That is a real finding about how specifications fail, not a complaint about a tool. Then you went one step further and explained why nothing downstream caught it: your Phase 4 memo reported only four summary values, and every one of them was either grid-invariant or coincidentally at the floor. The error had no way to surface. That is root-cause analysis.

| Criterion | Score |
|---|---|
| LLM execution & comparison | 25 / 25 |
| Hand verification | 25 / 25 |
| Recommendation & executive voice | 25 / 25 |
| Spec retrospective | 17 / 17 |
| Repo polish | 4.8 / 8 |
| **Total** | **97 / 100** |

**What you did well — and why it matters**

- **You separated real matches from coincidental ones.** The Put row at `n = −5` matches the workbook exactly, and a weaker analyst would have logged it as a tick. You labelled it **"Coincidental match"** and explained that both the stale and live `S_T` sit below `K_PUT`, so the floor binds either way. Distinguishing "agrees because it is right" from "agrees because the difference cannot show up here" is the actual skill this stage is testing, and almost nobody does it.
- **Your crossover numbers are correct — I recomputed both.** No-hedge overtakes the forward at `F0_in` itself, 1.165737, which is 1.149% above spot ✓. The put overtakes at `F0_in + FV_PREM/FC_AMT` = 1.165737 + 0.021854 = **1.187591**, which is 3.045% above spot — your "roughly 3.0%" ✓. Using the *future value* of the premium rather than the raw $0.021 is the subtle half, and you had it because you fixed that convention in Stage 2 and never drifted.
- **Your parity tie is honest about being tautological.** *"Parity check: USD_MM − USD_FWD ≈ $0 ✓ (tautological by construction, since F0_in is itself CIP-derived from the same R_USD/R_FC/S0_in… still true after Phase 4 since no live outright forward quote was ever sourced.)"* Most students report the parity match as though it validated something. You correctly identified it as a self-consistency check, not evidence, and named the precise condition — a sourced outright forward quote — that would make it a real test.
- **You defended the LLM where it deserved defending.** On the call option: *"The LLM's refusal to invent a premium was the more defensible choice given what it was handed."* You marked it spec ambiguity rather than LLM error, and then took the fix onto yourself in v2 point 2. Assigning fault to your own document when the evidence points there is the mark of a real post-mortem.
- **Your recommendation states its own losing case.** §D recommends the put, then immediately says that if the CFO's priority is budget certainty the forward is the better fit and *"either is acceptable."* You also priced the middle path. A memo that gives the CFO the decision rule rather than just the answer is what a treasurer actually wants.

**The one substantive correction — the $533,000 in §E is not an exposure number**

§E closes by weighing no-hedge against *"an average $461,000–$533,000 swing at a 5% move against us in this analysis."*

The $533,400 is not a swing. It is the LLM-vs-workbook delta from the `No Hedge / n = +5` row of your own comparison table — a diagnostic residual from the stale-grid error, which has migrated into the executive memo as though it measured exposure.

The actual swing is symmetric and easy to state: `FC_AMT × S0_in × 5%` = 8,000,000 × 1.1525 × 0.05 = **$461,000**, in either direction. That is why your §B range ($8,759,000–$9,681,000) is exactly ±$461,000 around $9,220,000. There is no $533,000 case at ±5%.

This matters more than a typo would, because it is the one number in the memo a CFO would repeat in a meeting, and the range implies an asymmetry the model does not contain. The exposure is linear in spot — that is the whole reason the no-hedge line is a diagonal. Replace the range with the single figure. While you are there, §A's *"$450,000 to over $850,000"* has the same drift at the top end: at ±9% the swing is 9,220,000 × 0.09 = $829,800, so "over $850,000" overstates your own worst case.

**One thing to think harder about — the ATM put is protecting from the wrong reference point**

You identify the mechanism correctly in §C: *"an at-the-money put only protects from today's spot, while the forward captures the full USD–EUR interest-rate carry."* The consequence is that recommending the ATM put costs $280,728.50 of guaranteed floor (9,325,898.13 − 9,045,169.63 ✓ — your figure is exact) *plus* $174,830 of premium, to buy upside that only begins paying above 1.1876.

That can still be the right call, and your mandate-based argument for it is legitimate. But the sharper version of your own analysis is to ask what strike makes the trade honest: a put struck at the *forward* rather than at spot would compare like-for-like against the forward's guarantee. It would cost more, and quantifying that is beyond what this stage asked for — but "the ATM strike is a choice, not a default" is the level your §C is already reasoning at, and stating it would close the gap between your very good sensitivity analysis and your recommendation.

**Repo polish — 3.2 points, and the easiest ones on the board**

`LICENSE` and the one-line repository description are both missing; those are the whole gap. Separately, as portfolio hygiene rather than grading: both Stage 5 filenames still carry the literal template braces — `2026-08-14-Badua-{pharma-exporter}-validation.md`. The placeholder was meant to be substituted, and `{` `}` in a path is the kind of thing that breaks tooling and looks unfinished to anyone browsing the repo. Rename both to `…-Badua-pharma-exporter-…` and update the cross-links in the memo header.

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
