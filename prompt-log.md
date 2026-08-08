# Prompt Log

A running record of the AI prompts I used across this project, updated every stage.
LLM = drafter, me = editor. I review and revise every output before committing it.

---

## Stage 0 — Portfolio Repository

### Bio (README.md)

**Prompt:**
> Help me draft a 150–200 word professional bio for my GitHub profile README.
> Background: BBA student at UH Mānoa (Management, HR Management, International Business),
> expected 2027, Shidler DAP completed Spring 2025. Barista at Honolulu Coffee Company
> (trained 5+ staff, wait times under 10 min), run my own e-commerce small business
> (100% on-time shipping, 10+ 5-star reviews), volunteer at Hawaiian Humane Society,
> member of SHRM and IBO clubs. Seeking an internship.
> Make it specific and quantified, not generic.

**What I edited:** Reviewed the draft for accuracy, adjusted tone to sound like me, and
confirmed every stat matches my real experience before committing.

### Resume (RESUME.md)

**Prompt:**
> Reformat my existing resume into clean Markdown for a GitHub repo. Keep all my real
> content, tighten the wording, and quantify accomplishments where possible.

**What I edited:** Checked dates, titles, and metrics against my actual resume; refined
bullet phrasing; verified nothing was invented.

---

*Future stages: add each new prompt below as the project continues.*

## Stage 1 — Executive Memo

**Prompt:**

> Draft a 300–400 word CFO memo using the decision-memo template for my FX hedging scenario: US tech services firm, EUR 12,500,000 receivable due in 1 year, indicative forward 1.0910. Cover the exposure, the USD risk of an adverse move, three hedge families (forward, money market, option) with honest pros/cons, and the Stage 2–5 roadmap.

**What I edited:** Reworded sections in my own voice, checked the numbers against my scenario parameters, and finalized before committing.

## Stage 2 — Model Specification

**Prompt:**

> Using the Stage 2 instructions, the official template-spec.md, and my scenario (#3 tech-services: EUR 12,500,000 receivable, 1 year, forward 1.0910, put premium $0.017, call premium $0.022, strikes and rates author-set), draft a 2–3 page technical spec covering all eight required sections in named-range notation with the ten standard names, the money-market hedge in three explicit steps, and concrete validation check figures.

**Iteration (gap found → fix):** The template's legacy calculation flow uses the textbook 1-year `(1 + r)` form with a single `BASIS` range, and the first draft carried that into §4/§5 — conflicting with the Stage 2 requirement to state the rate basis explicitly. I had the AI restate every growth factor as `1 + r × T_DAYS/360` (ACT/360, both legs), add ACT/360 to each rate row in §2, and then re-derive parity-consistent placeholder rates (S0 = 1.0700, R_USD = 4.00%, R_FC = 2.00%) so the §7 parity check actually passes: F_implied = 1.09127 vs. F0 = 1.0910; MM vs. forward gap = $3,325 (0.024%, inside the 0.05% tolerance).

**What I edited:** Trimmed template sections outside the required eight to hold 2–3 pages; verified all ten standard names appear exactly; confirmed §5 contains zero cell addresses; check figures V1–V10 were recomputed in Python rather than trusting the AI's arithmetic.

## Stage 3 — AI-Assisted Build + Audit

**Prompt:**

> Build the FX hedge workbook from my committed Stage 2 spec (docs/specs/2026-08-07-Patague-tech-services-spec.md), exactly as written: 8-tab architecture, the ten named ranges attached to the Inputs cells, every calculated cell a formula in named-range notation, MM hedge in three visible steps, formula-driven ±5% sensitivity grid with chart, and the §7 validation checks computed live with PASS/FAIL verdicts. The spec goes in as-is — no re-explaining in chat.

**Audit process:** Recalculated every formula (LibreOffice), read computed values back against the spec's §7 check figures, and ran scripted scans of named-range attachments and cell data types. Full findings in `analysis/2026-08-07-Patague-build-audit.md` (2 found-and-fixed, 2 checked-and-confirmed).

**Iteration (spec defect → fix):** The audit caught that spec v0.2 never specified the call payoff the build contract requires — the AI built exactly what the spec said, which is the point of the exercise. Fixed the spec (v0.3, §5.6), then regenerated the call block rather than patching ad hoc in chat. Also fixed two #VALUE! cells where documentation text starting with "=" was ingested as formulas. Each fix committed individually.
