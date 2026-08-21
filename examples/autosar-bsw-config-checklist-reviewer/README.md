# autosar-bsw-config-checklist-reviewer — example

**What this skill produces:** A compliance-audit checklist xlsx over an AUTOSAR Classic Basic Software configuration workbook (AUTOSAR R22-11) — **9 checks, `C001`–`C009`**, counted from `scripts/check_definitions.py` on 2026-08-21, spanning MCAL configuration, ECU Abstraction completeness, Service Layer criticality, OS task scheduling, memory allocation, NvM alignment, parameter conflicts, inter-module dependencies, and post-build variants. Output tabs: Title, Document Control, one tab per check group, References, plus the Summary dashboard (KPI tiles, rating pie, compliance bars, findings table).

**Typical input shape:** A completed BSW configuration xlsx — normally the output of `autosar-bsw-config-builder`, best-effort on other BSW-shaped workbooks. `scripts/bsw_probe.py` reads `Title` and `ECU-C Top-Level`, then the inventories: `MCAL Module Inventory` (name, configured), `ECU Abstraction Inventory` (name, depends_on), `Service Layer Modules` (name, critical), `Complex Drivers` (name, purpose), `Module Parameters` (module_id, parameter, value, post_build), `Schedule Tables` (name, period_ms, priority) and `Memory Map` (region, start, size_kb, purpose).

**Expected output:** `bsw_checklist.xlsx` (default name if no second argument) with every check rated FC / LC / PC / NO / NA and a recommended action attached to anything below FC. The source BSW workbook is never modified.

**Sample I/O:**

```bash
python scripts/generate_checklist.py bsw_config.xlsx output_checklist.xlsx
```

Do **not** trust a clean run here yet. Per [#54](https://github.com/jherrodthomas/automotive-skills-suite/issues/54), two of the nine checks are currently unreachable against real builder output: `C005` (memory allocation) rates `NA — "No memory regions defined"` because the builder silently drops `memory_layout`, and `C006` (NvM alignment) filters on a `module_id` column that never carries a module name. Both are Mandatory, so a passing dashboard on those two rows means "not checked", not "compliant". `scripts/recalc.py` in this archive is a 39-byte placeholder ([#55](https://github.com/jherrodthomas/automotive-skills-suite/issues/55)) — harmless today, since this reviewer emits no formula cells, but it is not the working `recalc.py` the other 146 skills ship.

**Paired builder:** `autosar-bsw-config-builder` (polished this week under #51, `15d4c8b`).

> **Doc drift found 2026-08-21 (DOCS run, not fixed):** `SKILL.md` advertises "~30 compliance checks" and the `check_definitions.py` module docstring repeats "(~30 checks)"; the file defines **9** (`C001`–`C009`, nine `check_*` functions). The claim is off by more than 3×. Also carries the mis-generated-batch `## Skills inventory` heading described in #55 — that heading holds trigger guidance, not an inventory. Both left unfixed deliberately: repacking a `.skill` zip is POLISH-mode work. The check-count correction should be folded into the #55 batch pass, which already touches this file's SKILL.md.
