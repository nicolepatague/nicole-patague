# Stage 5 Validation — Independent LLM Run, Comparison & Hand Verification

**Author:** Nicole Patague · **Date:** 2026-08-07
**Scenario:** #3 — U.S. Tech Services Firm (`tech-services`) · EUR 12,500,000 receivable, 365 days
**Workbook:** `models/builds/2026-08-07-Patague-tech-services-model.xlsx` (live-data run of 2026-08-07)
**Raw LLM output:** [`2026-08-07-Patague-llm-run-raw.md`](2026-08-07-Patague-llm-run-raw.md) (verbatim, with the logged prompt)

---

## Part 1 — Independent LLM execution

A **fresh LLM session** (clean context, no conversation history) was given **exactly two documents** as GitHub links — the Stage 2 spec and the Stage 4 market-data memo — and asked to compute all hedge outcomes at baseline and ±5% and recommend a strategy. No coaching, no mid-run corrections, no workbook results shared. The exact prompt is logged in the raw-output appendix and in `prompt-log.md`.

## Part 2.1 — Comparison table (LLM vs. workbook)

Settlement-date USD at three grid points. Workbook values from the recalculated live run.

| Strategy · S_T | LLM | Workbook | Match / diagnosis |
|---|---:|---:|---|
| No hedge · 1.095825 | $13,697,812.50 | $13,697,812.50 | ✓ exact |
| No hedge · 1.1535 | $14,418,750.00 | $14,418,750.00 | ✓ |
| No hedge · 1.211175 | $15,139,687.50 | $15,139,687.50 | ✓ |
| Forward (all S_T) | $14,581,250.00 | $14,581,250.00 | ✓ |
| MM step 1 / 2 / 3 | €12,144,877.05 / $14,009,115.67 / $14,581,524.25 | same to the cent | ✓ |
| Put · 1.095825 | $14,197,567.31 | $14,197,567.31 | ✓ (floor) |
| Put · 1.1535 | $14,197,567.31 | $14,197,567.31 | ✓ (kink) |
| Put · 1.211175 | $14,918,504.81 | $14,918,504.81 | ✓ |
| **Call · 1.095825** | **$13,411,576.08** | **$13,984,048.92** | **✗ — diagnosed below** |
| **Call · 1.1535** | **$14,132,513.58** | **$14,704,986.42** | **✗** |
| **Call · 1.211175** | **$15,574,388.58** | **$14,704,986.42** | **✗** |
| Validation checks V1–V8 | all PASS | all PASS | ✓ same computed values |

**Diagnosis of the call discrepancy — LLM error, enabled by a spec-presentation gap.** The LLM stated the spec "gives no call payoff formula in §5" and invented one by analogy: receivable proceeds plus a long-call payoff (`S_T×FC_AMT + MAX(0,S_T−K_CALL)×FC_AMT − FV_PREM_CALL`). The spec does define the call — §5.6, added at v0.3: `USD_CALL(S_T) = MIN(S_T,K_CALL)×FC_AMT + FV_PREM_CALL`, a **payable-side outlay**, not receivable proceeds. So the two numbers aren't even the same economic object (cost to obtain EUR vs. proceeds from selling EUR) — classified as an **LLM read error**, but one made likely by the spec: §5.6 sits after "No hedge," is labeled a "variant," and the outputs table doesn't mark direction (proceeds vs. outlay). Both models agree the premium carry is $286,236.42. Everything else — 9 of 10 named quantities, all three MM steps, every check figure — reproduced **to the cent** from the documents alone. The LLM's other two stated assumptions (full-precision grid values; live-derived check expectations per memo §4) match the workbook's actual behavior.

**LLM recommendation vs. mine:** the LLM recommended the money-market hedge on the frictionless numbers ($274.25 over the forward), itself caveating that transaction costs would erase the edge. My recommendation memo weighs that caveat the other way — see `docs/decisions/2026-08-07-Patague-tech-services-hedge-recommendation.md`.

## Part 2.2 — Hand verification (calculator + named-range notation, no Excel)

**1 · Forward proceeds** — `USD_FWD = FC_AMT × F0_in`
```
12,500,000 × 1.1665
  = 12,500,000 + (12,500,000 × 0.1665)
  = 12,500,000 + 2,081,250
  = $14,581,250.00          → workbook USD_FWD: $14,581,250.00 ✓
```

**2 · Money-market hedge, all three steps** (ACT/360; T/B = 365/360 = 1.0138889)
```
DF_FC  = 1 + 0.02884 × 1.0138889 = 1 + 0.0292406 = 1.0292406
DF_USD = 1 + 0.0403  × 1.0138889 = 1 + 0.0408597 = 1.0408597

Step 1  MM_BORROW  = 12,500,000 / 1.0292406        = €12,144,877.05
Step 2  MM_USD_NOW = 12,144,877.05 × 1.1535        = $14,009,115.67
Step 3  USD_MM     = 14,009,115.67 × 1.0408597     = $14,581,524.25
                                        → workbook USD_MM: $14,581,524.25 ✓
Parity: USD_MM − USD_FWD = 274.25 → 274.25 / 14,581,250 = 0.0019% < 0.05% ✓
```

**3 · Put outcome at S_T = 1.095825 (−5% — floor case)**
```
FV_PREM_PUT = 0.017 × 12,500,000 × 1.0408597
            = 212,500 × 1.0408597               = $221,182.69
S_T < K_PUT (1.095825 < 1.1535) → put exercised at K_PUT:
USD_PUT = 1.1535 × 12,500,000 − 221,182.69
        = 14,418,750.00 − 221,182.69            = $14,197,567.31
                              → workbook USD_FLOOR_PUT: $14,197,567.31 ✓
```

**4 · (extra) Call outlay at baseline** — `MIN(S_T,K_CALL)×FC_AMT + FV_PREM_CALL`
```
FV_PREM_CALL = 275,000 × 1.0408597 = $286,236.42
USD_CALL = 1.1535 × 12,500,000 + 286,236.42 = $14,704,986.42
                              → workbook USD_BASE_CALL: $14,704,986.42 ✓
```

All four hand computations reconcile to the workbook exactly.

## Spec retrospective (½–1 page)

**What the LLM got wrong or had to guess, and what that says about the spec:**

1. **It missed §5.6 entirely and invented a call payoff.** That is partly a read error, but the spec set the trap: the call section was bolted on at v0.3 (audit fix), placed *after* "No hedge" rather than beside the put in the option section, and labeled a "payable-side variant" easy to dismiss when analyzing a receivable. **v2 would** fold the call into §5.4 as "Option hedges — put (this exposure) and call (payable mirror)," and it wouldn't have been a v0.3 patch at all had the spec been written against the build contract's checklist from the start.
2. **Proceeds vs. outlay is never labeled.** The outputs table (§8) lists `USD_BASE_CALL` beside proceeds outputs with no direction column, so even a careful reader could plausibly interpret a "call outcome" as receivable proceeds — exactly the LLM's reading. **v2 would** add a direction/unit column (proceeds vs. outlay) to §8.
3. **Precision policy is implicit.** The memo displays grid points at 4 dp (1.0958) while the model computes full precision (1.095825); the LLM had to state an assumption. It chose correctly, but a spec shouldn't make it choose. **v2 would** state: "carry full precision; round display only."
4. **Two checks can't be verified from documents alone.** The LLM correctly flagged V9/V10 (no error cells; named-ranges-only) as workbook-structural and unverifiable from the spec + memo. **v2 would** tag each §7 check as *document-verifiable* or *workbook-structural* so a cold reader knows the boundary of what they can confirm.

**What held up:** the ten-name contract, ACT/360 statement, three-step MM decomposition, premium-carry rule, and computed check figures were strong enough that a cold model reproduced 9 of 10 quantities and every validation figure to the cent, and independently located the same winner-crossover (S_T ≈ 1.16652) the workbook shows. The documents mostly stood alone; the one failure traces to the one section that was patched in late rather than designed in.
