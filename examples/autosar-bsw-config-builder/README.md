# autosar-bsw-config-builder — Example

**What this skill produces:** An AUTOSAR Classic Basic Software configuration workbook per AUTOSAR
R22-11 — 14 tabs, verified against the generator on 2026-08-20: `Title`, `Document Control`,
`ECU-C Top-Level`, `MCAL Module Inventory`, `ECU Abstraction Inventory`, `Service Layer Modules`,
`Complex Drivers`, `Module Parameters`, `Post-Build Variants`, `Memory Map`, `Schedule Tables`,
`Inter-Module Dependencies`, `Validation Rules`, `References`. It captures the middleware stack for
one ECU across all four AUTOSAR Classic layers, together with module parameters, OS task scheduling
and the ECU-C top-level target definition.

**Typical input shape:** A single JSON file — `examples/sample_input.json` ships inside the skill
archive as a worked Body Control Module example. Keys: `item` (name, abbr, project, doc_id,
revision, date, author, approver, company); `ecu_target` (microcontroller, vendor, memory_ram_kb,
memory_rom_kb); `bus_interfaces[]` (name, type, baudrate, channels); `mcal_modules[]` (id, name,
configured_for_target); `ecu_abstraction[]` (id, name, depends_on); `service_layer[]` (id, name,
critical); `complex_drivers[]` (id, name, purpose); `module_parameters[]` (module_id, parameter,
value, post_build_variant); `memory_layout[]` (region, start_address, size_kb, purpose); and
`os_tasks[]` (id, name, period_ms, priority).

**Expected output:** `bsw_config.xlsx` (or whatever second argument you pass). Nine of the fourteen
tabs populate from input; five are header-only scaffolds the analyst completes by hand — see the
caveat below, which is the single most important thing to know before using this skill.

**Sample I/O:**

```bash
python scripts/generate_bsw.py examples/sample_input.json BCM-bsw.xlsx
```

prints `Generated BCM-bsw.xlsx` — a Body Control Module on an Infineon TC397XX with 8 MCAL modules,
3 ECU Abstraction modules, 8 Service Layer modules (5 marked critical), 2 complex drivers, 5 module
parameters across two post-build variants, and 5 OS tasks from 1 ms to 100 ms. Running the paired
reviewer against that file yields 9 checks and a Summary dashboard reporting 56% compliant,
0 major issues.

**Known caveat — two input fields are silently dropped ([#54](https://github.com/jherrodthomas/automotive-skills-suite/issues/54)):**
`memory_layout` and `bus_interfaces` are documented in the generator's schema and accepted without
error, but **neither reaches the workbook**. `Memory Map` comes out header-only even when you supply
regions, and bus interfaces appear nowhere at all. This matters beyond the empty tab: the paired
reviewer's `check_memory_allocation` (C005, Mandatory) then rates your supplied data
`NA — "No memory regions defined"`. Until #54 lands, fill `Memory Map` in by hand after generating.
`Post-Build Variants`, `Inter-Module Dependencies` and `Validation Rules` are likewise header-only
by design and expect manual completion.

**Paired reviewer:** `autosar-bsw-config-checklist-reviewer` — it resolves tabs by **exact name**
(`wb["MCAL Module Inventory"]`) and reads fixed column positions starting at row 3. Renaming a tab
or reordering a column breaks the probe silently and must be done in the same commit. The tab-name
contract was audited on 2026-08-20 and every name matches. Note that C006 (`check_nvm_alignment`)
matches on the Module ID column and so never fires against this builder's output; also tracked in
#54.
