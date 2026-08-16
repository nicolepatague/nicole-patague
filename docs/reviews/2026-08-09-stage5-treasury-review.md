# Stage 5 review — tech services LLM analysis & validation · Treasury sign-off

Nicole — this memo would survive a real treasury committee, and one sentence in §D is why: *"Our independent model-validation run reached the same frontier and picked MM on the frictionless numbers; the forward is the same trade without the frictions."* You disclosed that your independent check disagreed with your recommendation, explained the disagreement (the $274 money-market edge is smaller than the transaction costs your model excludes), and held your position anyway. Reporting a result that cuts against your own conclusion is the single hardest thing to do in analytical writing.

| Criterion | Score |
|---|---|
| LLM execution & comparison | 25 / 25 |
| Hand verification | 25 / 25 |
| Recommendation & executive voice | 25 / 25 |
| Spec retrospective | 17 / 17 |
| Repo polish | 6.4 / 8 |
| **Total** | **99 / 100** |

**What you did well — and why it matters**

- **Your two crossover rates are both correct.** I recomputed them. Unhedged overtakes the lock above 1.16652 — which is just `F0_in`, exactly right, since the forward's break-even is the forward rate by construction. The put needs 1.18422, which is `F0_in` plus the *carried* premium (221,182.69 / 12.5M = 0.01769). Using the carried premium rather than the raw $0.017 is the subtle part, and you got it because you set that convention in your Stage 2 spec and never drifted from it.
- **"Three shapes: the diagonal, the flat line, and the hockey stick."** That is the entire payoff geometry of FX hedging in nine words, and a CFO will remember it after the numbers are gone. Genuinely good executive writing.
- **You anticipated the accounting question.** "Cash-flow hedge designation under ASC 815 is available for a plain forward and can be confirmed with the auditors before execution." Nobody asked for this. It is exactly what the CFO would have asked next, and knowing that a plain vanilla forward gets clean hedge-accounting treatment — where an exotic structure might not — is a real consideration in instrument choice.
- **You costed the put in the language of the decision.** "Its floor is $383,683 below the forward lock, which is a high price for optionality after an 8% rally." That connects a number to a judgment about *when* in the cycle you are buying protection.

**The one thing to fix — "certainty here is unusually cheap" is not right**

§C concludes: "With the forward already locking a rate *above* today's spot, certainty here is unusually cheap: we give up only the upside beyond +1.13%."

The forward sits above spot for exactly one reason, and you named it correctly in §B: USD rates exceed EUR rates. That premium is not a discount on certainty — it is the arbitrage-free compensation for the interest differential, and under covered interest parity it is *always* there whenever `R_USD > R_FC`. It tells you nothing about whether hedging is cheap today versus last month.

The give-up-only-1.13% framing has the same problem. The break-even for a forward is the forward rate, always, by construction — that is true whether the forward trades at a premium, a discount, or flat. If EUR rates exceeded USD rates the forward would sit *below* spot and the same sentence would read "certainty is unusually expensive," when nothing about the economics would have changed. The two statements are the same statement.

This matters because "the forward is above spot, so hedging is cheap right now" is a real and common way desks talk themselves into sizing a hedge on carry rather than on exposure — which is a position, not a hedge. Your §E gets this right ("protect USD value, not speculate"); §C briefly argues the other way. Cut it or reframe it as what it is: the forward premium reflects the rate differential and is not evidence about the attractiveness of hedging.

Two smaller edits: **"a locked nine-figure-basis-point gain over plan"** in §E is not a real unit — it reads as a mid-edit collision. And **"$943,750 better than plan"** is a budget variance driven by the euro rallying 8% since you set the placeholder, not something the hedge earned; keep it, but label it a spot move rather than a hedging result.

**Repo polish — 1.6 points**

Only gap is a missing `LICENSE` file. Add one (MIT is fine for coursework) and you are at 100.

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
