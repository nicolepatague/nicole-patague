# Stage 3 review — tech services build & audit · Treasury sign-off

Nicole — Finding 2 is the best single piece of work I have read this term. You noticed the workbook had no call payoff, traced it to the *spec* rather than the build ("spec v0.2 defined `K_CALL`/`PREM_CALL` as inputs but specified no call calculation"), **fixed the spec first (v0.3, new §5.6), and only then rebuilt the workbook.** Fixing the upstream document before patching the downstream artifact is the difference between a model that stays correct and one that drifts from its own documentation. That is how this is supposed to work.

| Criterion | Score |
|---|---|
| Contract compliance | 50 / 50 |
| Structure & presentation | 25 / 25 |
| Audit note | 12.5 / 25 *(instructor-adjusted — see below)* |
| **Total** | **100 / 100** |

**A note on the grade.** The audit-note scanner counts findings by matching bulleted or numbered lists; your `## Finding N` headings return zero, which scored the criterion 12.5/25. I read the note by hand — four findings, two of them real defects found and fixed — and restored the full 12.5 points.

**What you did well — and why it matters**

- **You stated your method before your findings.** "LibreOffice recalculation of every formula; scripted read-back of computed values vs. spec check figures; scripted scan of named-range attachments and cell data types." A reader can now judge how much your clean result is worth. Almost nobody does this, and it is the first thing a real auditor is asked.
- **Finding 1 diagnoses a root cause, not a symptom.** `#VALUE!` at `Inputs!E11:E12` because the Stage-4 source text began with "=" and the generator ingested documentation as a formula. That is a genuinely subtle failure mode — a *comment* that Excel parsed as code — and you found it, explained it, fixed it, and tied it back to the spec rule it violated (V9).
- **You labelled findings by outcome.** "FOUND & FIXED" versus "CHECKED & CONFIRMED." Two words that let a reader instantly separate defects from confirmations. Steal-worthy.
- **Finding 3 verifies to the cent against pre-committed figures.** `F_implied` 1.091266 vs `F0_in` 1.0910; `USD_MM` $13,640,824.94 vs `USD_FWD` $13,637,500.00, a $3,324.94 gap = 0.0244%, inside your own 0.05% tolerance. You are checking against numbers you wrote down *before* building, which is the only way a check can honestly fail.
- **Finding 4 verified the winner labels at the boundaries.** Confirming that money-market wins in flat/down scenarios "by the $3,325 parity gap" and no-hedge wins at +5% shows you tested the *logic* of the comparison, not just that the cells populated.

**To push it further (real-desk nuance)**

- **Your $3,325 parity gap deserves a why, not just a pass.** It is 0.0244% — comfortably inside tolerance, so PASS is correct. But at Stage 4 that same gap fell to 0.0019% ($274). The Stage 3 gap was larger because your placeholder rates were parity-consistent only to rounding. Being able to say *why* a residual shrank is worth more than noting that both passed.
- **The call is a payable-side instrument in a receivable model.** You added `USD_CALL(S_T)` correctly and labelled it a "payable-side variant" — good. Make sure the Sensitivity tab and any chart keep it visually separate from the three receivable strategies, so no reader can misread the call column as a hedge candidate for this exposure.
- **157 calculated cells, zero hardcoded — verify that claim stays true.** You scripted the check once. Re-run that script after Stage 4 population; the most common way a formula-only workbook degrades is someone pasting a value in during a data update.

**Next — Stage 4**

Already in and reviewed separately. Your V3/V5/V7 checks had placeholder-era constants baked into them as *expected* values — you caught that at Stage 4 and it is the most instructive thing in that memo.

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
