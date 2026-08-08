<div style="border-top: 6px solid #024731; border-bottom: 1px solid #B2B2B2; padding: 12px 0; margin-bottom: 24px; font-family: 'Open Sans', Helvetica, Arial, sans-serif;">
  <div style="color: #024731; font-weight: 700; letter-spacing: 0.06em; text-transform: uppercase; font-size: 0.85rem;">University of Hawaiʻi at Mānoa · Shidler College of Business</div>
  <div style="color: #000000; font-weight: 700; font-size: 1.25rem; margin-top: 4px;">FIN-321 International Finance &amp; Securities</div>
  <div style="color: #525252; font-weight: 400; font-size: 0.95rem;">FX Transaction Hedging Project — Technical Specification</div>
</div>

# U.S. Tech Services Firm — FX Transaction Hedge Model · Technical Specification

> Technical specification for the FX transaction hedge model — the named-range contract, tab architecture, calculation flow, and validation checks, precise enough that an AI or a colleague who has never seen the Stage 1 memo could build the complete workbook from this document alone.

| Field | Value |
|------|------|
| **Created by** | Nicole Patague |
| **Date Created** | 2026-08-07 |
| **Version** | 0.2 |
| **LLM Used** | Claude (drafter; author as editor — see `prompt-log.md`) |
| **Role** | Treasury Analyst |
| **Audience** | CFO / Director of Treasury |
| **Scenario** | #3 — U.S. Tech Services Firm (`tech-services`) |

---

## 1. Problem Statement

The firm expects a **EUR 12,500,000 receivable settling in 365 days** from a European enterprise services contract. All rates in this model are quoted **USD per EUR**; a EUR depreciation over the horizon directly reduces realized USD proceeds and compresses contract gross margin. At the indicative spot of 1.0700 the receivable is worth ≈ $13.38M; each $0.01 decline in EURUSD costs the firm $125,000. This specification documents the workbook that quantifies and compares four strategies — **no hedge, forward, money-market, and option (put) hedge** — and produces the sensitivity evidence supporting the Stage 4 recommendation to the CFO. Objective: protect the USD value of the receivable while making the cost of preserving upside explicit.

---

## 2. Inputs — the Named-Range Contract

Every input is a workbook **named range**; §5 reads identically whether implemented in Excel, Python, or an AI prompt. All values below are **indicative — replaced with live market data at Stage 4**. Market inputs are the only cells an analyst adjusts for scenario work.

| Named range | Description | Placeholder | Unit | Stage-4 source |
|-------------|-------------|------------:|------|----------------|
| `FC_AMT` | Foreign-currency receivable | 12,500,000 | EUR | Contract terms (fixed) |
| `S0_in` | Spot rate at inception | 1.0700 | USD per EUR | ECB reference rate / Yahoo Finance EURUSD, close of Stage-4 pull date |
| `F0_in` | Forward rate, 1-yr maturity | 1.0910 | USD per EUR | Scenario sheet; Stage 4: spot + CME/Bloomberg 12-mo forward points |
| `R_USD` | USD interest rate | 4.00% | Annual %, ACT/360 | Fed H.15, 1-yr Treasury constant maturity |
| `R_FC` | EUR interest rate | 2.00% | Annual %, ACT/360 | 12-mo Euribor (EMMI) |
| `K_PUT` | Put strike (author-set at spot) | 1.0700 | USD per EUR | Set = Stage-4 `S0_in`, nearest listed CME strike |
| `K_CALL` | Call strike (author-set at spot) | 1.0700 | USD per EUR | Set = Stage-4 `S0_in`, nearest listed CME strike |
| `PREM_PUT` | Put premium per 1 EUR | 0.017 | USD | Scenario sheet; Stage 4: CME EUR/USD option quotes |
| `PREM_CALL` | Call premium per 1 EUR | 0.022 | USD | Scenario sheet; Stage 4: CME EUR/USD option quotes |
| `T_DAYS` | Days to settlement | 365 | Days | Contract settlement date − inception date |

Where the scenario leaves a value to the author (strikes, both interest rates), the placeholder chosen is stated above with how Stage 4 sources the real figure. Placeholder rates were chosen parity-consistent with `F0_in` (see §7, V1) so the model's internal checks are meaningful before live data arrives.

---

## 3. Tab Architecture

| Tab | Purpose |
|-----|---------|
| `Cover` | Title, author, version, date, scenario ID, one-paragraph model summary |
| `Legend_Key` | Cell-color legend (yellow inputs, blue assumptions, black formulas, green cross-tab links) and named-range glossary |
| `Inputs` | All §2 named ranges with units, sources, access dates — single source of truth; no calculations |
| `Fwd_Hedge` | Forward hedge calculation (§5.2) |
| `MM_Hedge` | Money-market hedge, three visible steps (§5.3) + parity check cell |
| `Opt_Hedge` | Put hedge: premium future-value and payoff (§5.4) |
| `Sensitivity` | 11-row `S_T` grid, four strategy columns, winner labels, comparison chart (§6) |
| `Notes_Assumptions` | §4 in full; data-source URLs and access timestamps; change log |

---

## 4. Assumptions & Constraints

- **Quote convention:** all rates USD per EUR; higher = EUR appreciation.
- **Day-count:** simple interest, **ACT/360 on both legs**: growth factor = `1 + r × T_DAYS/360`. One convention for both currencies is a deliberate simplification, noted on `Notes_Assumptions`.
- **Parity expectation:** the money-market hedge should replicate the forward within rounding (covered interest parity). A gap > 0.05% is treated as an input or formula error, not a finding (§7, V1–V2).
- **Premium treatment:** option premia are paid upfront in USD per 1 EUR (no contract multiplier), treated as negative cash at t₀, and **carried forward at `R_USD`** so all strategies are compared in settlement-date USD.
- **Transaction costs, bid-ask spreads:** excluded from base case; flagged as a Stage-4 sensitivity candidate.
- **Credit/counterparty risk, tax, hedge accounting:** excluded; model reports pre-tax cash outcomes.
- **Scenario construction:** `S_T` varied deterministically on a grid; no probability weights.

---

## 5. Calculation Flow

Named-range notation only — no cell addresses. Each numbered step below is its own cell in the build.

### 5.1 Derived inputs
1. `DF_USD = 1 + R_USD × T_DAYS/360` (growth factor)
2. `DF_FC = 1 + R_FC × T_DAYS/360`
3. `FV_PREM_PUT = PREM_PUT × FC_AMT × DF_USD` (premium cost at settlement)

### 5.2 Forward hedge
- `USD_FWD = FC_AMT × F0_in` — locked at t₀, invariant across the grid.

### 5.3 Money-market hedge — three explicit cells
1. **Borrow EUR today:** `MM_BORROW = FC_AMT / DF_FC` — sized so the receivable exactly repays the loan at settlement.
2. **Convert at spot:** `MM_USD_NOW = MM_BORROW × S0_in`.
3. **Invest in USD:** `USD_MM = MM_USD_NOW × DF_USD`.

### 5.4 Option hedge (put floor)
- For each settlement spot `S_T`: `USD_PUT(S_T) = MAX(S_T, K_PUT) × FC_AMT − FV_PREM_PUT`
- Equivalent decomposition (shown on `Opt_Hedge` for audit): `S_T × FC_AMT + MAX(0, (K_PUT − S_T)) × FC_AMT − FV_PREM_PUT`.
- Below `K_PUT` the floor holds; above it the put expires worthless and the firm sells at market — minus the premium either way.

### 5.5 No hedge
- `USD_NO_HEDGE(S_T) = S_T × FC_AMT`.

---

## 6. Sensitivity Plan

- **Grid:** `S_T_grid = S0_in × (1 + n × 1%)`, `n = −5 … +5` → 11 rows from `0.95×S0_in` (1.0165) to `1.05×S0_in` (1.1235), baseline row flagged at `S_T = S0_in`.
- **Columns per row:** `USD_NO_HEDGE`, `USD_FWD` (flat), `USD_MM` (flat), `USD_PUT`, hedge profit vs. no hedge per strategy, and a winner label = `ARGMAX` across the four.
- **Chart:** one line chart, `S_T` on x-axis, settlement-date USD proceeds on y-axis, four series.
- **What it lets the CFO see:** the flat forward/MM lines (certainty) crossing the no-hedge diagonal at ≈ `F0_in`; the put's kink at `K_PUT` — floored downside, participating upside, shifted down by the premium. The chart makes the price-of-certainty vs. price-of-optionality trade-off visible in one glance.

---

## 7. Validation Rules (Stage 3 Audit Checklist)

| # | Rule | Check figure (placeholders) |
|---|------|------------------------------|
| V1 | **Parity:** `F_implied = S0_in × DF_USD / DF_FC` must be within 0.25 cents of `F0_in` | `F_implied = 1.09127` vs. 1.0910 ✓ |
| V2 | `USD_MM` ties to `USD_FWD` within 0.05% | $13,640,825 vs. $13,637,500 → gap 0.024% ✓ |
| V3 | Forward proceeds | `USD_FWD = $13,637,500.00` exactly |
| V4 | MM steps | `MM_BORROW = €12,251,565` · `MM_USD_NOW = $13,109,175` · `USD_MM = $13,640,825` |
| V5 | Premium carry | `FV_PREM_PUT = $221,118` |
| V6 | **Kink:** at `S_T = K_PUT`, `USD_PUT = K_PUT × FC_AMT − FV_PREM_PUT = $13,153,882`; identical for all `S_T ≤ K_PUT` (floor) |
| V7 | Baseline no-hedge: at `S_T = S0_in`, `USD_NO_HEDGE = $13,375,000` |
| V8 | Grid symmetric around `S0_in`; exactly 11 rows; step = 1% of `S0_in` |
| V9 | **No error cells** anywhere in the workbook; every output cell is a formula (no pasted values) |
| V10 | Every formula references named ranges only — zero bare cell addresses |

---

## 8. Outputs

| Named output | Description | Location |
|--------------|-------------|----------|
| `USD_FWD` | Forward proceeds (scalar) | `Fwd_Hedge`, summary block |
| `USD_MM` | Money-market proceeds (scalar) | `MM_Hedge`, summary block |
| `PARITY_GAP` | `USD_MM − USD_FWD`, $ and % | `MM_Hedge` |
| `USD_FLOOR_PUT` | `MIN(USD_PUT)` across grid — worst case | `Opt_Hedge` |
| `USD_BASE_PUT` | `USD_PUT` at `S_T = S0_in` | `Opt_Hedge` |
| `SENS_TABLE` | 11×6 sensitivity table (§6) | `Sensitivity` |
| `SENS_CHART` | Four-series comparison line chart | `Sensitivity` |
| `SUMMARY_TBL` | Per-strategy baseline proceeds + floor, executive at-a-glance | `Cover` |

All summary results are gray-filled cells named exactly as above; the Stage 3 grading script and Stage 4 prompt read these names.

---

## Appendix A — Change Log

| Version | Date | Author | Change |
|---------|------|--------|--------|
| 0.1 | 2026-08-07 | N. Patague | Initial AI draft from template + scenario 3 |
| 0.2 | 2026-08-07 | N. Patague | Fixed rate basis to ACT/360 on both legs; re-derived parity-consistent placeholder rates; added check figures V1–V10 |
