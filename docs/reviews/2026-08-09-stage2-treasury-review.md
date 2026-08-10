# Stage 2 review — pharma exporter · Treasury sign-off

Allan Jay — one line in your decision context does more work than most whole specs: *"No house view on EUR direction has been formed, so no family is being recommended ahead of Phase 4 live pricing."*

That is a Treasury analyst refusing to prejudge the answer before the data arrives. It also protects you at Stage 5 — you have written down, in advance, that the recommendation must follow from pricing rather than from a currency opinion you formed in week one. Most students quietly decide on the forward at Stage 2 and spend three stages confirming it.

| Criterion | Score |
|---|---|
| Named-range contract & tab architecture | 30 / 30 |
| Calculation flow | 30 / 30 |
| Validation & sensitivity plan | 20 / 20 |
| Reproducibility & prompt log | 20 / 20 |
| **Total** | **100 / 100** |

**What you did well — and why it matters**

- **You sized the exposure before proposing to hedge it.** "An ordinary one-year currency swing (±5–9%) can move realized USD proceeds by roughly $400,000–$800,000, flowing directly into USD revenue forecasting, cash planning, and covenant headroom." That is the paragraph that justifies the whole project existing. Naming *covenant headroom* in particular is the detail that turns an FX question into a financing question — a translation loss that breaches a covenant is a materially worse event than the same loss in isolation.
- **Your forward placeholder is derived, not invented.** `F0_in` 1.1136 is marked "indicative, derived via covered interest parity — see §4, Step 1." That single choice means your Stage 3 parity check will have a real target and your money-market and forward legs will reconcile. Students who pick a round number for the forward and unrelated round numbers for the rates discover at Stage 3 that their validation fails on inputs they made up.
- **You specified the spot source to the quote level.** "Bloomberg BGN spot mid, timestamped at model build" — not "Bloomberg." Naming the *composite* and the *side* (mid, not bid or ask) is the difference between a source and a reproducible instruction.
- **You named your role and audience.** Treasury Analyst / FP&A writing to CFO / Director of Treasury. And `FC_AMT`'s Stage-4 source is the "sales contract / AR aging schedule" rather than a market feed — correctly recognizing that the notional comes from the accounting system, not the market.
- **You versioned the document and dated both creation and update.** v0.1 with created/updated fields. Keep incrementing it as the spec changes at Stages 3 and 4; a spec that never changes version is usually a spec nobody went back to.

**To push it further (real-desk nuance)**

- **Show the parity arithmetic inline, not just the reference.** You point to §4 Step 1 for the derivation of 1.1136. Put the substituted numbers on the page — `S0_in × (1 + R_USD×T/360) / (1 + R_FC×T/360)` with your actual placeholders — so a reader can confirm the forward is parity-consistent without cross-referencing. It is one line and it makes the spec self-contained, which is the standard you set for yourself in the opening blockquote.
- **Pre-commit the parity tolerance numerically.** How close must the money-market result sit to the forward before you call it a pass — 0.05% of notional, $1,000, a basis point on the rate? Deciding before you see the answer is what stops the tolerance from being sized to whatever the model produces.
- **Watch the tenor on both rate legs at Stage 4.** Your horizon is 365 days, so both `R_USD` and `R_FC` need one-year instruments — a 1-year Treasury CMT (FRED `DGS1`) and 12-month Euribor or the ECB one-year euro-area yield curve point. The common trap is grabbing an overnight policy rate because it is quoted daily; an overnight rate in a one-year carry calculation is the most frequent silent error at Stage 4.
- **The put is not the only way to keep upside.** Your objective says "retain upside if EUR strengthens." Worth adding a collar to the comparison — financing the put by selling a call caps the upside but cuts the premium, and on a EUR 8M notional the premium saving is real money. It gives the CFO a third shape rather than a binary.

**Next — Stage 3**

Build straight from §2 and your calculation flow, then audit against your own validation rules. The bar is at least three *substantive* findings — and an audit where every check returns PASS is weaker than one that finds a real defect, because it usually means the tests were not hard enough. Go looking for what is wrong.

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
