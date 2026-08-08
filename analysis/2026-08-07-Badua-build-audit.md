# Build Audit — pharma-exporter Model

Workbook audited: `models/builds/2026-08-07-Badua-pharma-exporter-model.xlsx`
Spec: `docs/specs/2026-08-07-Badua-pharma-exporter-spec.md`
Audited by: Allan Jay Badua · 2026-08-07

Method: `recalc.py` (LibreOffice) run after each build pass to force real evaluation, then every named range, formula, and check-figure cell inspected directly with `openpyxl` (formula view and `data_only=True` value view) rather than trusting a visual skim.

---

## Finding 1 — Descriptive text notes were silently parsed as formulas, causing 6 `#N/A` errors

**Checked:** Ran `recalc.py` immediately after the first generation pass, before doing anything else.

**Found:** 6 errors, all `#N/A`, all in `Inputs!F13:F21` — the "Source / Notes" column. Six annotation cells (e.g. explaining `DF_USD = 1 + R_USD × T_DAYS / BASIS_USD` in plain language) had been written as strings starting with `"= 1 + R_USD × ..."`. Excel/LibreOffice treats any cell value beginning with `=` as a formula regardless of intent, and `×` / `−` aren't valid operators, so every one of those six cells failed to evaluate.

**Fixed:** Reworded all six to start with `"Note: ..."` instead of `"= ..."`, rebuilt, and reran `recalc.py`. Result: `status: success`, `total_errors: 0` across 145 formulas.

---

## Finding 2 — All ten required named ranges confirmed attached to the correct cells

**Checked:** Dumped `wb.defined_names` from the saved workbook and cross-referenced each target cell against the Inputs tab layout.

**Found:** `FC_AMT`, `S0_in`, `F0_in`, `R_USD`, `R_FC`, `K_PUT`, `K_CALL`, `PREM_PUT`, `PREM_CALL`, and `T_DAYS` all resolve to the intended `Inputs!` cells (e.g. `F0_in → Inputs!$C$15`, `K_CALL → Inputs!$C$19`). Twelve additional supporting names (`DF_USD`, `USD_FWD`, `USD_MM`, `USD_FLOOR_PUT`, `USD_CEILING_CALL`, etc.) also resolve correctly and are used throughout the Forward, MoneyMarket, Options, and Sensitivity tabs instead of bare cell references.

**Fixed:** Nothing — this one was clean. Flagging it explicitly because "everything passed" is exactly the kind of claim that needs the underlying check shown, not just asserted.

---

## Finding 3 — The parity check is currently tautological, not a live test

**Checked:** Whether `MoneyMarket!C14` (`USD_MM − USD_FWD`) is a meaningful test of covered interest-rate parity.

**Found:** It computes to exactly **$0.00**, not merely "within 0.05%." That's because `F0_in` is itself derived on the Inputs tab as `S0_in × DF_USD / DF_FC` — the same parity relationship the money-market walk uses to build `USD_MM`. So the check currently proves the two formulas are algebraically identical, not that real quoted market rates satisfy parity. That's expected and correct for an indicative, internally consistent spec, but it's worth being honest that this check has no teeth yet.

**Fixed:** Added a cell comment on `Inputs!C15` (`F0_in`) and a line in the Notes-Validation tab flagging this explicitly, so that at Phase 4 — when `F0_in` is replaced by a live dealer forward quote instead of a derived figure — the parity check becomes a real, non-trivial test instead of a formality. No structural change made now: hardcoding `F0_in` today just to make the check "interesting" would violate the formulas-only requirement.

---

## Finding 4 — Spec's hand-rounded base case doesn't quite tie to the workbook's precision

**Checked:** Compared the spec's §7.1 base-case table (`USD_FWD ≈ $8,908,800`) against the workbook's computed value.

**Found:** The workbook returns **$8,908,478.16** — a ~$322 gap. The spec rounded `F0_in` to 1.1136 by hand before multiplying by `FC_AMT`; the workbook carries `DF_USD`/`DF_FC` at full floating-point precision through the same formula. Not a workbook defect — the workbook is the more accurate of the two.

**Fixed:** No change to the workbook. Documenting here that the spec's §7.1 table should be read as directionally illustrative, and the workbook (Sensitivity + Notes-Validation tabs) is now the authoritative source of record for these figures going forward.

---

## Finding 5 — `K_PUT` is intentionally not formula-linked to `S0_in`

**Checked:** Whether `K_PUT` moves automatically when `S0_in` changes, since the spec describes it as "defaulted to `S0_in`, at-the-money."

**Found:** `K_PUT` (`Inputs!C16`) is a hardcoded yellow input, not a formula referencing `S0_in`. This is deliberate — auto-linking the strike to spot would mean the strike silently redefines itself every time someone updates the spot quote, which a treasury desk does not want. But it does mean a Phase-4 analyst who updates `S0_in` with a live quote must remember to update `K_PUT` separately, or the "at-the-money" framing quietly goes stale.

**Fixed:** Confirmed the existing cell comment and Source/Notes entry on `Inputs!C16` already state this explicitly ("defaults to `S0_in`; override manually"), so the risk is documented rather than hidden. No formula change — flagging the failure mode was the fix.

---

## Summary

| # | Area | Status |
|---|---|---|
| 1 | Formula errors from text notes starting with `=` | Found & fixed |
| 2 | All 10 named ranges attached correctly | Confirmed clean |
| 3 | Parity check currently tautological (F0_in self-referential) | Found, documented, not "fixed" — deferred to Phase 4 by design |
| 4 | Spec vs. workbook rounding gap (~$322 on USD_FWD) | Found & documented; workbook is source of truth |
| 5 | K_PUT not auto-linked to S0_in | Confirmed intentional; documented the failure mode |

Final `recalc.py` result: `status: success`, `total_errors: 0`, `total_formulas: 145`.
