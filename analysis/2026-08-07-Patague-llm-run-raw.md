# Stage 5 — Raw Independent LLM Run (Appendix)

**Run:** 2026-08-07 · fresh LLM session (clean context, no conversation history)
**Documents provided (exactly two, as GitHub links):** the Stage 2 spec and Stage 4 market-data memo.
**No coaching:** no corrections mid-run, no workbook results pasted. Output below is verbatim.

## Prompt (logged)

> You are a treasury analyst. Fetch and read these two documents, and nothing else:
>
> 1. https://raw.githubusercontent.com/nicolepatague/nicole-patague/main/docs/specs/2026-08-07-Patague-tech-services-spec.md
> 2. https://raw.githubusercontent.com/nicolepatague/nicole-patague/main/data/2026-08-07-Patague-market-data.md
>
> Using only what these two documents specify, compute the complete FX hedge analysis and recommend a strategy. Specifically:
>
> - Compute USD outcomes for each strategy: no hedge, forward hedge, money-market hedge (show all three steps), option (put) hedge, and the call variant — at the baseline settlement spot and at the −5% and +5% points of the sensitivity grid the spec defines.
> - Show your arithmetic and state every value you use and where it came from in the documents.
> - Run the validation checks the spec defines and report pass/fail.
> - If anything is ambiguous or missing, state the assumption you made rather than asking.
> - Finish with a recommendation of one hedge strategy for the firm, justified from your computed numbers.
>
> Return the full analysis as markdown.

## Raw output (verbatim)

All data needed is in the two documents. Here is the complete analysis, computed from the live Stage-4 inputs (per the market-data memo, live values supersede the spec's placeholders; the spec supplies all formulas and conventions).

---

# FX Transaction Hedge Analysis — U.S. Tech Services Firm (Scenario #3)

**Exposure:** EUR 12,500,000 receivable, settling in 365 days (spec §1). Risk: EUR depreciation reduces USD proceeds.

## 1. Inputs used and provenance

| Input | Value | Source |
|---|---:|---|
| `FC_AMT` | EUR 12,500,000 | Spec §2 / memo §1 (contract, unchanged) |
| `S0_in` | 1.1535 USD/EUR | Memo §1 — ECB reference fix, 2026-08-07 |
| `F0_in` | 1.1665 USD/EUR | Memo §1/§3 — CIP-implied (no free live quote) |
| `R_USD` | 4.03% | Memo §1 — FRED DGS1 (1-yr Treasury CMT), obs. 2026-08-05 |
| `R_FC` | 2.884% | Memo §1 — 12-mo Euribor, fixing 2026-08-06 |
| `K_PUT` = `K_CALL` | 1.1535 | Memo §1 — set at live spot per scenario convention |
| `PREM_PUT` | $0.017/EUR | Memo §1 — scenario-sheet assumption, kept |
| `PREM_CALL` | $0.022/EUR | Memo §1 — scenario-sheet assumption, kept |
| `T_DAYS` | 365 | Spec §2 |
| Day count | ACT/360, simple interest, both legs | Spec §4 |
| Premium treatment | Paid at t₀, carried forward at `R_USD` | Spec §4 |

## 2. Derived inputs (spec §5.1)

- `DF_USD = 1 + 0.0403 × 365/360 = 1 + 0.0408597 = 1.0408597`
- `DF_FC = 1 + 0.02884 × 365/360 = 1 + 0.0292406 = 1.0292406`
- `FV_PREM_PUT = 0.017 × 12,500,000 × 1.0408597 = 212,500 × 1.0408597 = $221,182.69`
- `FV_PREM_CALL = 0.022 × 12,500,000 × 1.0408597 = 275,000 × 1.0408597 = $286,236.42`

## 3. Sensitivity points (spec §6)

Grid: `S_T = S0_in × (1 + n × 1%)`, n = −5…+5. The three points requested:

- **−5%:** S_T = 1.1535 × 0.95 = **1.095825**
- **Baseline:** S_T = S0_in = **1.1535**
- **+5%:** S_T = 1.1535 × 1.05 = **1.211175**

## 4. Strategy computations

### 4.1 No hedge (§5.5): `S_T × FC_AMT`
- −5%: 1.095825 × 12.5M = **$13,697,812.50**
- Base: 1.1535 × 12.5M = **$14,418,750.00**
- +5%: 1.211175 × 12.5M = **$15,139,687.50**

### 4.2 Forward hedge (§5.2): `FC_AMT × F0_in` — invariant
- 12,500,000 × 1.1665 = **$14,581,250.00** at all three points.

### 4.3 Money-market hedge (§5.3) — three steps, invariant
1. **Borrow EUR today:** `MM_BORROW = 12,500,000 / 1.0292406 = €12,144,877.05`
2. **Convert at spot:** `MM_USD_NOW = 12,144,877.05 × 1.1535 = $14,009,115.67`
3. **Invest in USD:** `USD_MM = 14,009,115.67 × 1.0408597 = $14,581,524.25` at all three points.

(All three steps match the memo's workbook and FX Hedging Lab cross-check exactly.)

### 4.4 Put hedge (§5.4): `MAX(S_T, K_PUT) × FC_AMT − FV_PREM_PUT`
- −5%: MAX(1.095825, 1.1535) = 1.1535 → 14,418,750.00 − 221,182.69 = **$14,197,567.31** (floor holds)
- Base: MAX(1.1535, 1.1535) = 1.1535 → **$14,197,567.31** (kink point, V6)
- +5%: MAX(1.211175, 1.1535) = 1.211175 → 15,139,687.50 − 221,182.69 = **$14,918,504.81**

### 4.5 Call variant
**Assumption (stated, not asked):** the spec defines `K_CALL`/`PREM_CALL` as inputs (§2) but gives no call payoff formula in §5 — the calculation flow covers the put only. I computed the call by direct analogy to the §5.4 decomposition: `USD_CALL(S_T) = S_T × FC_AMT + MAX(0, S_T − K_CALL) × FC_AMT − FV_PREM_CALL` (receivable sold at market plus long-call payoff, premium carried at `R_USD`). Note that for a *receivable*, a long call is not a hedge — it adds upside leverage with no floor; the strike/premium pair most likely exists so the same template serves payable scenarios.

- −5%: 13,697,812.50 + 0 − 286,236.42 = **$13,411,576.08**
- Base: 14,418,750.00 + 0 − 286,236.42 = **$14,132,513.58**
- +5%: 15,139,687.50 + (1.211175 − 1.1535) × 12.5M − 286,236.42 = 15,139,687.50 + 720,937.50 − 286,236.42 = **$15,574,388.58**

## 5. Results table (settlement-date USD; profit vs. no hedge in parentheses)

| S_T | No hedge | Forward | Money-market | Put | Call variant | Winner (4 spec strategies) |
|---|---:|---:|---:|---:|---:|---|
| 1.095825 (−5%) | 13,697,812.50 | 14,581,250.00 (+883,437.50) | **14,581,524.25** (+883,711.75) | 14,197,567.31 (+499,754.81) | 13,411,576.08 (−286,236.42) | Money-market |
| 1.1535 (base) | 14,418,750.00 | 14,581,250.00 (+162,500.00) | **14,581,524.25** (+162,774.25) | 14,197,567.31 (−221,182.69) | 14,132,513.58 (−286,236.42) | Money-market |
| 1.211175 (+5%) | **15,139,687.50** | 14,581,250.00 (−558,437.50) | 14,581,524.25 (−558,163.25) | 14,918,504.81 (−221,182.69) | 15,574,388.58 (+434,701.08) | No hedge |

Crossover points (from the numbers): no-hedge overtakes MM at S_T > 14,581,524.25 / 12.5M = **1.16652** (+1.13%; first grid row above it is +2%, matching the memo's "winner flips MM → no-hedge at ≈ +2%"). The put overtakes MM only above S_T = (14,581,524.25 + 221,182.69) / 12.5M = **1.18422** (+2.66%).

## 6. Validation checks (spec §7), run on live inputs

| # | Check | Computed | Result |
|---|---|---|---|
| V1 | `F_implied = S0 × DF_USD/DF_FC` within 0.25¢ of `F0_in` | 1.1535 × 1.0408597 / 1.0292406 = 1.16652 vs 1.1665 → gap 0.00002 | **PASS** |
| V2 | `USD_MM` ties `USD_FWD` within 0.05% | 14,581,524.25 − 14,581,250.00 = $274.25 = 0.0019% | **PASS** |
| V3 | Forward proceeds = `FC_AMT × F0_in` exactly | $14,581,250.00 | **PASS** (live-derived expectation, per memo §4 structural fix) |
| V4 | MM steps | €12,144,877.05 / $14,009,115.67 / $14,581,524.25 | **PASS** (matches memo and Lab cross-check) |
| V5 | Premium carry | $221,182.69 | **PASS** |
| V6 | Kink: at S_T = K_PUT, `USD_PUT` = K×FC_AMT − FV_PREM = $14,197,567.31; identical for all S_T ≤ K | −5% row and baseline both $14,197,567.31 | **PASS** |
| V7 | Baseline no-hedge = `S0 × FC_AMT` | $14,418,750.00 | **PASS** |
| V8 | Grid symmetric, 11 rows, 1% steps of S0 | 1.095825 → 1.211175, step 0.011535, baseline centered | **PASS** |
| V9 | No error cells; all outputs formulas | Workbook-structural; memo §4 reports 0 errors across 164 formulas | **PASS (as reported; not independently verifiable from documents)** |
| V10 | Named ranges only, no bare addresses | Workbook-structural; this analysis uses named-range notation throughout | **PASS (as reported)** |

**Assumptions made where documents were silent/ambiguous:** (1) call-variant payoff formula, as stated in §4.5; (2) V3/V5/V7 expected values recomputed from live inputs, following the memo's documented structural fix rather than the spec's placeholder-era constants; (3) the −5%/+5% spot values carried at full precision (1.095825/1.211175) rather than the memo's 4-dp display (1.0958/1.2112).

## 7. Recommendation

**Hedge with the money-market hedge** (borrow €12,144,877 today, convert to $14,009,116 at spot, invest at 4.03%), locking in **$14,581,524.25**.

Justification from the computed numbers:

- **It is the highest guaranteed outcome.** $14,581,524 beats the forward by $274.25 (CIP holds almost exactly — V2 gap 0.0019%) and beats the put's floor by **$383,957** (14,581,524 − 14,197,567). At the −5% grid point it preserves **$883,712** that the unhedged position loses.
- **The put's upside is expensive here.** The put only outperforms MM above 1.18422 — a further +2.66% EUR rally on top of a euro that has already rallied 7.8% from the 1.07 planning level — and its certain cost is $221,183 in carried premium at every point at or below the strike.
- **Remaining unhedged wins only above 1.16652** (+1.13%), and the spec's own framing (§1: each $0.01 decline costs $125,000; objective is to *protect* the USD value) argues against betting on further appreciation.
- **The call variant is unsuitable for this exposure:** it is the worst strategy at −5% and baseline ($286,236 certain premium drag, no floor) and only wins in strong EUR-appreciation scenarios — it adds risk rather than hedging it.

Caveat for the CFO: the $274 MM-over-forward edge is smaller than realistic transaction costs and bid-ask spreads (excluded from the base case per spec §4), and the MM hedge consumes borrowing capacity. If operational simplicity or credit-line usage matters, the **forward at 1.1665 ($14,581,250)** is economically equivalent; on the pure numbers as modeled, the money-market hedge is the recommendation.
