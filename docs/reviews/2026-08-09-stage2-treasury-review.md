# Stage 2 review — tech services · Treasury sign-off

Nicole — one line in §2 tells me you understood the assignment better than the assignment stated it: *"Placeholder rates were chosen parity-consistent with `F0_in` so the model's internal checks are meaningful before live data arrives."* Most students pick round placeholder numbers and then discover at Stage 3 that their parity check fails on inputs they invented. You reasoned backwards from "my validation has to be able to tell me something" and chose placeholders accordingly. That is design, not compliance.

| Criterion | Score |
|---|---|
| Named-range contract & tab architecture | 30 / 30 |
| Calculation flow | 30 / 30 |
| Validation & sensitivity plan | 20 / 20 |
| Reproducibility & prompt log | 20 / 20 |
| **Total** | **100 / 100** |

**What you did well — and why it matters**

- **You versioned the spec.** "Version: 0.3" with a change log on `Notes_Assumptions`. A spec is a living contract; a reader who finds v0.3 knows there was a v0.2 and can ask what changed. This pays off immediately at Stage 3 — your audit cites "spec v0.2 omitted the call payoff" and records the bump to v0.3. That is real configuration management.
- **You pre-committed the premium convention.** "Carried forward at `R_USD` so all strategies are compared in settlement-date USD." Comparing an upfront premium against settlement-date proceeds without carrying it is one of the most common errors in this whole project, and you closed it in the spec before the build. It also turned out to be the exact source of your Stage 4 lab discrepancy — which you were then able to explain rather than panic about, precisely because you had written the convention down.
- **You wrote a tolerance with a decision rule attached.** "A gap > 0.05% is treated as an input or formula error, not a finding." That sentence tells the Stage 3 auditor what to *do*, not just what to measure. A tolerance without a consequence is decoration.
- **You quantified the exposure in the reader's units.** "Each $0.01 decline in EURUSD costs the firm $125,000." A CFO does not think in pips; they think in dollars. One sentence converts the whole problem into their language.
- **You named the audience and your own role.** Treasury Analyst writing to CFO / Director of Treasury. Knowing who you are writing as and who is reading disciplines everything else in the document.

**To push it further (real-desk nuance)**

- **One day-count on both legs is a simplification you should keep visible.** You flagged ACT/360 on both currencies as deliberate — good. Be aware it is not market convention: EUR money-market instruments are ACT/360, but USD Treasury yields quote on a bond basis, and a real desk would not mix them silently. Your disclosure is the right treatment for a course model; just know the shortcut you are taking and why it is acceptable here (both legs consistent → the parity check remains internally valid).
- **`K_PUT = K_CALL = S0_in` is a choice worth defending.** Both at-the-money is a clean convention, but it means your put and call are not a natural collar — they are two separate at-the-money positions. When you get to the recommendation, be explicit that these are alternatives being compared, not a structure being proposed.
- **Say what the sensitivity grid is *for*.** You specify 11 rows, four strategies, winner labels, and a chart. Add one line on the management question it answers — "at what settlement rate does the decision change?" — so the exhibit has a thesis, not just a shape.

**Next — Stages 3, 4, 5**

All three are in and reviewed separately. Your Stage 3 audit found two real defects and your Stage 4 lab reconciliation is the strongest in the cohort — read those reviews next.

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
