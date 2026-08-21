# fmeda-checklist-reviewer — example

**What this skill produces:** A confirmation-review checklist xlsx over an ISO 26262-5 FMEDA workbook — **36 checks** across 7 tabs, counted from `scripts/check_definitions.py` on 2026-08-21: Title, General Info, Guide, Summary (KPI tiles, rating pie, compliance bars, findings table), **Confirmation Review (18 `cr_*` checks,** ISO 26262-8 §10 doc quality**)**, **FMEDA Assessment (14 `fmedaa_*` checks,** ISO 26262-5 §8, including the three automatic metric-target checks**)**, and **Verification Assessment (4 `va_*` checks,** all external Verification Plan / Report**)**.

**Typical input shape:** A completed FMEDA workbook xlsx — normally the output of `fmeda-builder`, best-effort on other FMEDA-shaped workbooks. `scripts/fmeda_probe.py` reads `00_Title_Page` (title, project, doc ID, revision, status, author, approver, ASIL), `02_Assumptions` (mission profile, FIT source, environmental class), `03_HW_Element_Catalog` (element ID, type, FIT, safety-related flag), `06_FMEDA_Worksheet` (failure-mode rows, distribution %, classification, allocated safety mechanism), and the computed values on `07_SPFM_Calculation`, `08_LFM_Calculation`, `09_PMHF_Calculation`.

**Expected output:** `<name>_checklist.xlsx` with every check rated FC / LC / PC / NO / NA, or left `Pending` with an `(Auto-suggest)` draft finding where reviewer judgment is required. The source FMEDA is never modified — gaps land in the Recommended Actions column.

**Sample I/O:**

```bash
python scripts/generate_checklist.py fmeda.xlsx output_checklist.xlsx
python scripts/recalc.py output_checklist.xlsx      # optional; detects formula errors
```

The three metric checks (`fmedaa_08` SPFM, `fmedaa_09` LFM, `fmedaa_10` PMHF) are the ones worth reading first. The probe opens the workbook with `data_only=True`, so it reads *evaluated* cell values and a formula-driven FMEDA is fully supported — but that also means a workbook written by openpyxl with no LibreOffice recalc pass presents as *empty* metrics rather than as failing ones. Run `recalc.py` against the source FMEDA first if the metric checks come back unexpectedly `NO`. All 4 VA checks returning `Pending` is the normal, correct result when no external Verification Plan is supplied — not a finding.

**Paired builder:** `fmeda-builder`. Upstream chain: `tsc-builder` → `fmeda-builder` → this reviewer; the builder-side TSC reader was repaired this week under #53 (`ec8fc8e`), so safety-mechanism allocations now actually reach the FMEDA worksheet that `fmedaa_07` checks.

> **Doc drift found 2026-08-21 (DOCS run, not fixed):** `SKILL.md` says "each of 14 + 14 checks" in Step 2, and its 7-tab output table labels the Confirmation Review tab "14 generic doc-quality checks". `check_definitions.py` registers **18** `CR` entries in `CHECKS` (IDs 1, 2, 3, 4a–4e, 5 …), so the true totals are **18 CR + 14 FMEDAA + 4 VA = 36**, not 28. Same drift class as `cs-architecture-checklist-reviewer` (42 → 43, recorded in the v2026.08.W33 known issues). Left unfixed deliberately: correcting it means repacking a `.skill` zip, which is POLISH-mode work, not a Friday docs change.
