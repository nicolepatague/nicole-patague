# Stage 4 review — tech services market data & population · Treasury sign-off

Nicole — your lab cross-check is the strongest in the cohort, and the reason is that it *failed*. Six quantities tied and one did not: the lab's put baseline read $14,206,250 against your workbook's $14,197,567.31. You did not adjust your model to match, and you did not wave it off as rounding. You computed the difference ($8,682.69), showed it equals the interest carry on the premium ($212,500 × 4.086%), identified the cause as a convention difference — the lab subtracts the undiscounted premium, your workbook carries it forward at `R_USD` per spec §4 — verified the explanation held on a second row, and then **kept your treatment and documented why**. That is a reconciliation. Everything else in this memo is good; this is the part that is professional.

| Criterion | Score |
|---|---|
| Data quality & provenance | 50 / 50 |
| Model resolves cleanly | 33 / 33 |
| Lab cross-check | 17 / 17 |
| **Total** | **100 / 100** |

**What you did well — and why it matters**

- **You caught a structural defect that only live data could expose.** Checks V3/V5/V7 had their *expected* values hard-coded to placeholder-era constants ($13,637,500 / $221,118.06 / $13,375,000), so they failed the instant real numbers loaded. Your diagnosis is exactly right — "expectations should derive from inputs" — and the fix (`FC_AMT*F0_in`, `PREM_PUT*FC_AMT*(1+R_USD*T_DAYS/360)`, `S0_in*FC_AMT`) makes the checks self-updating. A validation that hard-codes what it expects is not a control; it is a snapshot that will one day lie to you.
- **You chose a borrowing rate for a borrowing leg.** 12-month Euribor over a German Schatz yield, "consistent with the money-market hedge's EUR-borrow leg (a government Schatz yield would understate the borrowing cost)." Most students reach for the government curve on both legs because it is easier to source. Matching the *instrument* to the cash flow, not just the tenor, is a genuinely sophisticated call.
- **You explained your as-of dates rather than hiding the lag.** R_USD observed 2026-08-05 because "the H.15 release lags the calendar day"; R_FC fixed 2026-08-06 because "Euribor publishes T+0 ~11:00 CET and the 2026-08-07 fixing was not yet mirrored at retrieval." Anyone auditing your file will wonder why three inputs carry three different dates. You answered before they asked.
- **You decomposed the forward gap into its two moving parts.** The +0.0755 change versus the indicative forward is "almost entirely the spot move," while the forward *premium* actually narrowed from +1.96% to +1.13% as the rate differential compressed. Separating a spot effect from a carry effect is real FX analysis, not bookkeeping.

**To push it further (real-desk nuance)**

- **Your parity check is now nearly tautological — say so.** The gap fell to 0.0019% at Stage 4 largely because `F0_in` is CIP-implied from the same `S0_in`, `R_USD`, and `R_FC` that feed the money-market leg. Both legs are built from one input set, so agreement is close to guaranteed. It confirms your implementation, not the market. The genuine unknown — cross-currency basis plus dealer spread — never enters the model, because no quoted forward does.
- **You have the one quoted number you need — use it.** Your spec named CME 12-month forward points as the Stage 4 source for `F0_in`. Even one real quote compared against your 1.16652 would convert the parity check from an internal-consistency test into a market test, and the gap would *be* the basis. That is the single highest-value addition left in this model.
- **Quantify your data sensitivity.** At a one-year tenor a 25bp error in `R_FC` moves the CIP forward by roughly 0.0028 — about $35,000 on EUR 12.5M. One sentence like that tells a CFO how much your source quality is worth in dollars.

**Next — Stage 5**

Already in and reviewed separately.

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
