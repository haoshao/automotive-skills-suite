# Polish log — autosar-bsw-config-builder

## 2026-08-20 (POLISH, issue [#51](https://github.com/jherrodthomas/automotive-skills-suite/issues/51))

**Severity: high** (two documented input fields silently discarded; two Mandatory reviewer checks
structurally unreachable). First pass on this skill and the first pass on the `autosar` domain.

### What's good

- **The generator runs and the pair closes.** `generate_bsw.py` executed from a real input produces a
  clean 14-tab workbook, and `autosar-bsw-config-checklist-reviewer` probes that workbook, runs its
  9 checks and writes a dashboard without a single exception. Verified end-to-end from the repacked
  `.skill` archives, not from a working tree.
- **The builder-to-reviewer tab contract is intact.** Every sheet name `bsw_probe.py` looks for —
  `Title`, `ECU-C Top-Level`, `MCAL Module Inventory`, `ECU Abstraction Inventory`,
  `Service Layer Modules`, `Complex Drivers`, `Module Parameters`, `Schedule Tables`, `Memory Map` —
  is emitted by the builder under exactly that name, and the column positions line up too. This is
  the shape that broke in #43 and #53; here it is correct. Worth recording, since #46 deferred
  builder-to-reviewer pairs as unaudited.
- **All 14 sheet names are Excel-legal.** Longest is 25 chars (`ECU Abstraction Inventory`,
  `Inter-Module Dependencies`), none contain the illegal set. No repeat of the SOTIF problem.
- **Frontmatter is valid.** `name` and `description` both present, description 354 chars, well under
  the 1024 limit.
- **The reference material is accurate.** `references/bsw_modules.md` and `references/methodology.md`
  are correct AUTOSAR Classic content and correctly layered (MCAL / ECU Abstraction / Service /
  Complex Drivers).

### What to fix

**1. `memory_layout` and `bus_interfaces` are accepted and silently discarded — HIGH.**

The module docstring documents both fields in the input schema. Neither reaches the workbook.
`build_memory_map(wb)` takes no data argument at all, and `bus_interfaces` is never read by any
builder function. Verified with a sample declaring 4 memory regions and 3 bus interfaces:

```
INPUT   memory_layout: 4 regions      OUTPUT  Memory Map          data rows = 0
INPUT   bus_interfaces: 3 interfaces  OUTPUT  (absent from all 14 tabs)
```

A grep for `RAM_BSW` and `CAN1` across every cell of the output returns nothing. This is the #53
failure shape — no exception, no warning, just an empty tab that looks like the analyst forgot to
fill it in.

**2. The silent drop launders into a passing audit — HIGH.** This is the part that makes (1) more
than cosmetic. The reviewer's `check_memory_allocation` reads:

```python
if not probed.memory_regions:
    return CheckResult("NA", "No memory regions defined")
```

So an analyst who *did* supply memory layout gets check **C005 — Mandatory — rated NA, "No memory
regions defined"**, and the dashboard counts it as not-applicable rather than as a finding. The tool
reports the analyst's own supplied data as absent, on a mandatory check.

**3. `check_nvm_alignment` is structurally dead — MED.** It filters
`"NvM" in p.get("module_id")`, but `module_id` holds the *identifier* (`SL005`), never the module
name. The builder never writes a module name into that column, so C006 (Mandatory) can only fire if
someone hand-types `NvM` into the Module ID cell. Against the builder's own output it is permanently
NA. Confirmed: sample carries `NvMDatasetSelectionBits` under `SL005` and C006 still returned
`NA — No NvM parameters found`.

**4. Four of 14 tabs are permanently header-only — MED.** `Post-Build Variants`, `Memory Map`,
`Inter-Module Dependencies` and `Validation Rules` have no data path at all; their build functions
take only `wb`. `Post-Build Variants` is the sharpest case: `module_parameters` already carries a
`post_build_variant` field per row, so the data to populate the tab is present in the input and just
never aggregated. SKILL.md advertises all four as workbook content.

**5. Cosmetic/consistency, lower priority.**
- Data rows get no borders, no alternating fill, no column widths. `BORDER_ALL`, `ALT_ROW` and
  `LIGHT_BLUE` are defined and never applied below the header row; `get_column_letter` is imported
  and unused. For a workbook whose description claims "audit-ready", columns still render at
  default width.
- `build_document_control` writes a change row with `Author` and `Description` blank.
- Several tabs declare a header column the writer never fills (`Port Count` on MCAL,
  `Complexity` on ECU Abstraction, `Features` on Service Layer, `BSW Interaction` on Complex
  Drivers, `Unit`/`Notes` on Module Parameters).
- The reviewer's output sheets are named `Tab_MCAL`, `Tab_DEPS`, `Tab_SVC_LAYER` — internal
  identifiers surfacing as user-visible sheet names, out of step with the builder's human-readable
  tabs and with the rest of the repo. Reviewer-side, noted here for the pair.

### Applied this pass

Small and bounded, per the POLISH rule. All verified by regenerating from the repacked archive.

1. **Shipped `examples/sample_input.json`** (new). The skill had no sample input at all, which made
   DoD item 2 — "smoke-tested from its own sample input" — literally unsatisfiable; the only way to
   run it was the hard-coded one-module fallback inside `main()`. The sample exercises every
   documented schema field (8 MCAL modules, 3 ECU abstraction, 8 service layer, 2 complex drivers,
   5 parameters, 4 memory regions, 3 bus interfaces, 5 OS tasks) and is deliberately written to
   include the two fields that get dropped, so the defect is reproducible from a shipped artifact.
   Matches the `examples/sample_input.json` convention used by `cdd-builder` and `hara-builder`.
2. **Restored `scripts/recalc.py`** from the 39-byte placeholder to the repo-standard 5,782-byte
   utility, copied byte-identical from `autosar-swc-builder`. Its only dependency,
   `scripts/office/soffice.py`, was already shipped. Import verified. **Honest scope note:** this
   builder emits zero formula cells, so this is a consistency and latent-defect fix, not an active
   bug — the recalc step simply had nothing to recalculate. 146 of 152 skills ship the real file;
   see #55 for the other five.
3. **Fixed a wrong path in the References tab.** It pointed readers at
   `scripts/references/methodology.md`; the file actually ships at `references/methodology.md`.
4. **Corrected the `## Skills inventory` heading** in SKILL.md to `## When to use`. The section
   never contained an inventory — it holds trigger guidance. Template artifact, see #55.
5. **Documented the sample in SKILL.md** with the exact invocation line.

### Deliberately not done

Findings 1-4 are behaviour changes, not typos, so they are filed rather than fixed —
[#54](https://github.com/jherrodthomas/automotive-skills-suite/issues/54) for the dropped inputs and
dead checks, [#55](https://github.com/jherrodthomas/automotive-skills-suite/issues/55) for the
template batch. Wiring `memory_layout` is genuinely a six-line change that mirrors the existing
builder functions, but it changes what the workbook asserts and pairs with a reviewer-side fix to
C005/C006, so it deserves its own pass with its own verification rather than a drive-by.

### Verification performed

- Builder run from `examples/sample_input.json` -> 14 tabs, 0 Excel-illegal names, expected row
  counts on all populated tabs.
- Reviewer run against that output -> 9 checks, dashboard written, no exceptions.
- Both executed after repacking and reinstalling the `.skill` archive, so the archive itself is
  verified, not just a loose working copy.
- Input-vs-output diff on `memory_layout` and `bus_interfaces` to confirm the drop.
- `recalc.py` import check.

### Follow-ups

- Fix #54: wire `memory_layout` into `build_memory_map`, aggregate `post_build_variant` into
  `Post-Build Variants`, and key `check_nvm_alignment` on module *name*. Builder and reviewer must
  land together or C005 flips from NA to a false FC on the wrong column.
- Fix #55 for the remaining five files of the mis-generated batch.
- `bus_interfaces` has no tab to land in. Either add one or drop it from the documented schema —
  a schema that documents a field nothing consumes is worse than no schema.
