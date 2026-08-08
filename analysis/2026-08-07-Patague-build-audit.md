# Stage 3 Build Audit — FX Hedge Workbook

**Author:** Nicole Patague · **Date:** 2026-08-07
**Workbook:** `models/builds/2026-08-07-Patague-tech-services-model.xlsx`
**Audited against:** `docs/specs/2026-08-07-Patague-tech-services-spec.md` §7 (V1–V10), Stage 3 build contract
**Method:** LibreOffice recalculation of every formula; scripted read-back of computed values vs. spec check figures; scripted scan of named-range attachments and cell data types.

---

## Finding 1 — FOUND & FIXED: `#VALUE!` errors on Inputs (violates V9)

**Checked:** full-workbook recalculation (139 formulas at generation).
**Found:** `Inputs!E11` and `Inputs!E12` returned `#VALUE!`. Root cause: the Stage-4 source text for `K_PUT`/`K_CALL` began with "=" ("= Stage-4 S0_in…"), so the generator ingested documentation text as a formula. Violates spec V9 (no error cells).
**Did:** rewrote both cells as plain text ("Set to Stage-4 S0_in; nearest listed CME strike"); recalc then reported 0 errors. Commit: `Audit fix 1`.

## Finding 2 — FOUND & FIXED: spec omitted the call payoff (spec defect, build contract #5)

**Checked:** build contract #5 — "put and call with premium cost in USD and proceeds as a function of S_T" — against spec §5.
**Found:** spec v0.2 defined `K_CALL`/`PREM_CALL` as inputs but specified no call calculation, so the generated workbook had none. Per Stage 3 guidance this is a spec defect, not a chat-prompt problem.
**Did:** updated the spec first (v0.3, new §5.6: `USD_CALL(S_T) = MIN(S_T, K_CALL) × FC_AMT + FV_PREM_CALL`, payable-side variant), then added to `Opt_Hedge`: `FV_PREM_CALL` ($286,152.78), baseline outlay `USD_BASE_CALL` ($13,661,152.78), and an 11-row `USD_CALL(S_T)` grid mirroring the sensitivity spot ladder. Verified the cap binds above `K_CALL` (outlay constant at $13,661,153 for S_T ≥ 1.07) and floats below it ($12,992,403 at S_T = 1.0165). Commit: `Audit fix 2`.

## Finding 3 — CHECKED & CONFIRMED: parity and all §7 check figures pass to the cent

**Checked:** the seven live validation verdicts on `Notes_Assumptions` plus the parity block on `MM_Hedge`, after recalculation.
**Found:** all PASS. `F_implied` = 1.091266 vs. `F0_in` = 1.0910 (V1); `USD_MM` = $13,640,824.94 vs. `USD_FWD` = $13,637,500.00 — gap $3,324.94 = 0.0244%, inside the 0.05% tolerance (V2); MM steps €12,251,565.48 → $13,109,175.06 → $13,640,824.94 (V4); `FV_PREM_PUT` = $221,118.06 (V5); put kink value $13,153,881.94 and floor identical for all S_T ≤ K_PUT (V6); baseline no-hedge $13,375,000 (V7); grid symmetric, 11 rows (V8). Matches spec §7 exactly.
**Did:** no change required; documented as confirmation evidence.

## Finding 4 — CHECKED & CONFIRMED: named-range contract and formulas-only compliance

**Checked:** (a) dumped all defined names and compared each attachment to the Inputs layout — all ten contract names (`FC_AMT`…`T_DAYS`) point at the correct cells, plus 13 derived/output names per spec §8; (b) scripted scan of all 157 calculated cells across six tabs asserting each is stored as a formula.
**Found:** zero hard-coded values in calculation areas; zero bare cell references in hedge logic (all named-range notation); winner labels behave correctly at the boundaries (money-market wins in flat/down scenarios by the $3,325 parity gap; no-hedge wins at +5%; put participates upside net of premium).
**Did:** no change required.

---

**Residual notes for Stage 4:** placeholder inputs (S0 1.0700, R_USD 4.00%, R_FC 2.00%) remain flagged indicative; sources and access timestamps to be recorded on `Notes_Assumptions` when live data replaces them.
