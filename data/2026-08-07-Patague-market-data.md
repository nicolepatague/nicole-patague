# Stage 4 Market-Data Memo — Live Inputs & Provenance

**Author:** Nicole Patague · **Retrieved:** 2026-08-07 (afternoon HST)
**Workbook:** `models/builds/2026-08-07-Patague-tech-services-model.xlsx`
**Purpose:** provenance for every live input. An auditor should be able to re-pull each number from the source and date given.

---

## 1. Input provenance table

| Named range | Live value | Source | As-of / retrieved | Notes |
|-------------|-----------:|--------|-------------------|-------|
| `S0_in` | **1.1535** | ECB euro foreign-exchange reference rate (ecb.europa.eu, eurofxref-daily.xml) | Fix of 2026-08-07 (~16:00 CET) · retr. 2026-08-07 | USD per EUR, official daily reference fix |
| `R_USD` | **4.03%** | FRED series DGS1 — 1-yr U.S. Treasury constant maturity (H.15) | Obs. 2026-08-05 (latest posted; FRED updated Aug 6) · retr. 2026-08-07 | See rate rationale §2 |
| `R_FC` | **2.884%** | 12-month Euribor (EMMI), via euribor-rates.eu | Fixing of 2026-08-06 (latest published) · retr. 2026-08-07 | See rate rationale §2 |
| `F0_in` | **1.1665** | **CIP-implied — computed, not quoted** (§3) | Computed 2026-08-07 from the three inputs above | No free live 1-yr EURUSD forward quote available |
| `K_PUT` | **1.1535** | Set at live spot per scenario convention | 2026-08-07 | = `S0_in` |
| `K_CALL` | **1.1535** | Set at live spot per scenario convention | 2026-08-07 | = `S0_in` |
| `PREM_PUT` | **$0.017** | Scenario sheet — **kept, assumption** | — | Retail option quotes unreliable; per Stage 4 instructions |
| `PREM_CALL` | **$0.022** | Scenario sheet — **kept, assumption** | — | Same |
| `FC_AMT` | 12,500,000 EUR | Scenario #3 (tech-services) | — | Unchanged |
| `T_DAYS` | 365 | Scenario | — | Unchanged |

## 2. Rate choices — rationale

**`R_USD` = 1-yr Treasury CMT (DGS1, 4.03%).** A default-free USD benchmark at exactly the model's 1-year horizon, published in the Fed's H.15 release — the same source family named in the spec. Chosen over SOFR-term rates for simplicity and public auditability. Latest posted observation (2026-08-05) used; the H.15 release lags the calendar day.

**`R_FC` = 12-mo Euribor (2.884%).** The euro money-market reference rate at the matching tenor, and a *borrowing/deposit* rate — consistent with the money-market hedge's EUR-borrow leg (a government Schatz yield would understate the borrowing cost). Latest published fixing (2026-08-06); Euribor publishes T+0 ~11:00 CET and the 2026-08-07 fixing was not yet mirrored at retrieval.

## 3. CIP-implied forward — computation shown

No reliable free 1-year EURUSD outright forward quote was available, so `F0_in` is CIP-implied (per the Stage 4 instructions, this is how forwards are priced, stated explicitly):

```
F0 = S0 × (1 + R_USD × T/360) / (1 + R_FC × T/360)
   = 1.1535 × (1 + 0.0403 × 365/360) / (1 + 0.02884 × 365/360)
   = 1.1535 × 1.0408597 / 1.0292406
   = 1.16652   →  entered as 1.1665
```

**Gap vs. the scenario's indicative forward (1.0910): +0.0755 (+6.9%).** Almost entirely the spot move — the euro rallied from the 1.07-area placeholder to 1.1535 (+7.8%). The forward *premium* actually narrowed: indicative implied ≈ +1.96% (4.00% vs 2.00% placeholder rates) vs. live +1.13% (4.03% vs 2.884%), because the USD–EUR rate differential compressed. Direction and magnitude are both economically sensible.

## 4. Population & re-check

Live values entered **through the named-range input cells only** (`Inputs!C7:C12`). All formulas recalculated; 0 errors across 164 formulas. Live outputs:

| Output | Placeholder run | Live run |
|--------|----------------:|---------:|
| `USD_FWD` | $13,637,500.00 | **$14,581,250.00** |
| `USD_MM` | $13,640,824.94 | **$14,581,524.25** |
| Parity gap (V2) | 0.0244% | **0.0019%** — PASS |
| `F_implied` (V1) | 1.09127 | **1.16652** vs 1.1665 — PASS |
| `FV_PREM_PUT` | $221,118.06 | **$221,182.69** |
| `USD_BASE_PUT` / floor | $13,153,881.94 | **$14,197,567.31** |
| Sensitivity grid | 1.0165–1.1235 | **1.0958–1.2112**, winner flips MM → no-hedge at S_T ≈ +2% |

**Structural fix (documented per instructions):** checks V3/V5/V7 on `Notes_Assumptions` had their *expected* values hard-coded to placeholder-era constants ($13,637,500 / $221,118.06 / $13,375,000), so they failed the moment live data loaded. That was a structural defect — expectations should derive from inputs. Fixed by replacing the constants with independent recomputations (`FC_AMT*F0_in`, `PREM_PUT*FC_AMT*(1+R_USD*T_DAYS/360)`, `S0_in*FC_AMT`). All seven checks now PASS with live data.

## 5. FX Hedging Lab cross-check

Entered the live inputs (rates as 4.03 / 2.884 %) into the FX Hedging Lab (adamwstauffer.github.io/ai-lms/fxlab.html), 2026-08-07:

| Quantity | Lab | Workbook | Match |
|----------|----:|---------:|-------|
| Forward hedge | $14,581,250 | $14,581,250.00 | ✓ exact |
| MM step 1 (borrow) | €12,144,877 | €12,144,877.05 | ✓ |
| MM step 2 (convert) | $14,009,116 | $14,009,115.67 | ✓ |
| MM step 3 (invest) | $14,581,524 | $14,581,524.25 | ✓ |
| Implied forward | 1.1665 | 1.16652 | ✓ |
| MM − Forward | $274 | $274.25 | ✓ |
| Put @ baseline | $14,206,250 | $14,197,567.31 | **✗ — resolved below** |

**Discrepancy resolution:** the lab subtracts the *undiscounted* premium (0.017 × 12.5M = $212,500 → $14,206,250 at every grid point below/at strike). The workbook carries the premium forward at `R_USD` per spec §4 (`FV_PREM_PUT` = $221,182.69 → $14,197,567.31). Difference = $8,682.69 = exactly the interest carry on the premium ($212,500 × 4.0860%). Verified on the lab's +1% row as well (consistent with the undiscounted treatment). Neither model is broken: this is a premium-carry convention difference. The workbook keeps the spec §4 treatment (settlement-date comparability across strategies) and this note records why the lab reads ~$8.7K higher on the put column.

---
*AI assistance for the data hunt is logged in `prompt-log.md` (Stage 4 entry).*
