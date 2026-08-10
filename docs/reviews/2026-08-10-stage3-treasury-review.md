# Stage 3 review — pharma exporter build & audit · Treasury sign-off

Allan Jay — Finding 3 is the most sophisticated observation in this cohort's Stage 3 work, and you made it *about your own model*:

> "It computes to exactly **$0.00**, not merely 'within 0.05%.' That's because `F0_in` is itself derived on the Inputs tab as `S0_in × DF_USD / DF_FC` — the same parity relationship the money-market walk uses to build `USD_MM`. So the check currently proves the two formulas are algebraically identical, not that real quoted market rates satisfy parity."

A passing validation is the easiest thing in the world to accept. You looked at a check returning a perfect zero and asked whether that zero *meant* anything — and correctly concluded it did not. Then, rather than hardcoding `F0_in` to manufacture an interesting result, you added a cell comment and a Notes-Validation line so the check becomes a real test at Phase 4 when a live quote replaces the derived figure. Your reasoning for not "fixing" it is exactly right: "hardcoding `F0_in` today just to make the check 'interesting' would violate the formulas-only requirement."

| Criterion | Score |
|---|---|
| Contract compliance | 50 / 50 |
| Structure & presentation | 25 / 25 |
| Audit note | 12.5 / 25 *(instructor-adjusted — see below)* |
| **Total** | **100 / 100** |

**A note on the grade.** The audit-note scanner counts findings by matching bulleted or numbered lists; your `## Finding N` headings return zero, which scored the criterion 12.5/25. I read the note by hand — four findings, one a real defect found and fixed — and restored the full 12.5 points.

**What else you did well**

- **Your method paragraph names the tools and the reason.** `recalc.py` after each build pass "to force real evaluation," then `openpyxl` in both formula view and `data_only=True` value view, "rather than trusting a visual skim." A reviewer can now judge exactly how much your clean result is worth.
- **Finding 1 diagnoses a subtle root cause.** Six `#N/A` errors in `Inputs!F13:F21` because annotation strings began with `"= 1 + R_USD × ..."` — Excel treats any value starting with `=` as a formula, and `×`/`−` are not operators. A *documentation* cell parsed as code. You reworded to `"Note: ..."`, rebuilt, and confirmed 0 errors across 145 formulas.
- **Finding 2 shows its work even though nothing was wrong.** Your own justification is the point: "Flagging it explicitly because 'everything passed' is exactly the kind of claim that needs the underlying check shown, not just asserted." You also listed the twelve supporting names beyond the required ten and confirmed they resolve.
- **Finding 4 correctly identifies which artifact is wrong.** The spec's hand-rounded `USD_FWD ≈ $8,908,800` against the workbook's $8,908,478.16 — a ~$322 gap traced to rounding `F0_in` to 1.1136 by hand before multiplying, where the workbook carries full precision. Your conclusion that "the workbook is the more accurate of the two" is right, and knowing which of two disagreeing sources to trust is the actual skill.

**To push it further (real-desk nuance)**

- **Your workbook filename still carries the template placeholder.** `models/builds/2026-08-07-Badua-{scenario-slug}-model.xlsx` — the literal braces were never replaced with `pharma-exporter`. Your audit note references the correct name, so this is purely a filing slip, but it is the kind of thing that makes automated tooling skip your file (it did — my roster scanner treats `scenario-slug` as an unfilled template and dropped it). There is also a duplicate, `...-model (2).xlsx`. Rename the real one and delete the copy.
- **Findings 1 and 4 are defects; 2 and 3 are confirmations.** That is a good ratio. Keep Finding 3's habit — asking what a passing check actually proves — as your default at Stage 5.

**Next — Stage 4**

Already in and reviewed separately. Your Phase 4 memo does something almost nobody managed: it derives `PREM_CALL` from `PREM_PUT` by put-call parity instead of carrying two independent assumptions.

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
