# Hedge Recommendation — EUR 12.5M Receivable

**To:** Chief Financial Officer
**From:** Nicole Patague, Treasury Analyst
**Date:** 2026-08-07
**Re:** Recommended hedge for the EUR 12,500,000 receivable settling 2027-08-07
**Version:** 1.0 · **LLM Used:** Claude (drafter; author as editor — see `prompt-log.md`)
**Basis:** live market data of 2026-08-07 (`data/2026-08-07-Patague-market-data.md`); model validated by hand and by independent LLM run (`analysis/2026-08-07-Patague-tech-services-validation.md`)

---

## A · Exposure summary

We are owed EUR 12,500,000 in one year from European services contracts. At today's spot (1.1535), that is $14.42M; every $0.01 decline in EURUSD costs us $125,000. Since we framed this project the euro has already rallied roughly 8% from the 1.07 planning level — today's decision is as much about *banking* that windfall as protecting against reversal.

## B · Hedge outcomes (live data, settlement-date USD)

**Unhedged (baseline):** $14,418,750 at today's spot — but ranging $13,697,813 to $15,139,688 across ±5%. A −5% move surrenders $720,937 of value; nothing bounds it there.

**Forward at 1.1665:** locks **$14,581,250** — $162,500 *above* today's spot value (the euro trades at a forward premium because USD rates exceed EUR rates), and $943,750 above the $13.64M we used for planning at the indicative 1.0910. Zero upfront cost.

**Money-market hedge:** borrow €12,144,877 today, convert to $14,009,116, invest at 4.03% → **$14,581,524**. Ties the forward within $274 (0.0019%) — covered interest parity holds almost exactly in our live data — but consumes borrowing capacity and takes three transactions instead of one.

**Put at 1.1535 ($0.017/EUR premium):** guaranteed floor of **$14,197,567** net of the $221,183 carried premium, with full participation above the strike (e.g., $14,918,505 at +5%). The premium is a sunk cost in every scenario.

## C · Sensitivity interpretation

Under **EUR depreciation**, the forward/MM lock preserves $883,438–$883,712 versus unhedged at the −5% point; the put preserves $499,755. Under **EUR appreciation**, unhedged overtakes the lock only above 1.16652 (+1.13% from here), and the put needs more than +2.66% (above 1.18422) before it beats the lock — its premium is, in effect, a bet on a further euro rally on top of one that has already happened. The choice is between three shapes: the diagonal (full exposure), the flat line (certainty), and the hockey stick (a floor, for a fee). With the forward already locking a rate *above* today's spot, certainty here is unusually cheap: we give up only the upside beyond +1.13%.

## D · Recommendation

**Enter a 12-month forward to sell EUR 12,500,000 at 1.1665, locking $14,581,250.**

The money-market hedge is economically identical (+$274 on paper) but that edge is smaller than the transaction costs and bid-ask spreads our model deliberately excludes, and it ties up credit capacity we don't need to spend — so the forward's single-instrument simplicity wins the tie. The put is a reasonable alternative only if the board holds a strong view of continued EUR strength; its floor is $383,683 below the forward lock, which is a high price for optionality after an 8% rally. (Our independent model-validation run reached the same frontier and picked MM on the frictionless numbers; the forward is the same trade without the frictions.)

## E · Executive justification

**Budget certainty:** the forward fixes FY-27 revenue from this contract at $14.58M — $943,750 better than plan — removing FX from the forecast entirely. **Cash flow & liquidity:** no upfront cash (vs. $212,500 premium today for the put), no draw on borrowing lines (vs. the MM hedge), one instrument with one settlement. **Optionality forgone:** we surrender gains beyond 1.1665; given the objective set in our framing memo — protect USD value, not speculate — that is the point of the hedge, not a defect. **Cost:** zero premium; the implicit cost is the upside beyond +1.13%, which we consider well exchanged for a locked nine-figure-basis-point gain over plan. Hedge-accounting treatment (cash-flow hedge designation under ASC 815) is available for a plain forward and can be confirmed with the auditors before execution.

**Requesting approval to execute the forward this week while the 1.1665 level holds.**

---

*Figures: workbook `models/builds/2026-08-07-Patague-tech-services-model.xlsx` (live run 2026-08-07, all validation checks passing); hand-verified in `analysis/2026-08-07-Patague-tech-services-validation.md`.*
