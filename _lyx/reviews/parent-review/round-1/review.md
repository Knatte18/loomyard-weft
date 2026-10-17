---
verdict: APPROVED
---

The design does what the board entry's operator decision of 2026-10-17 asks, both steps and no more.

Step 1, per-segment settings: D1 replaces the per-row-name maps with `Env.Segments map[string]SegmentSettings` keyed by segment, so a new resolved value is one struct field and a new review segment is one map entry; D2 gives the constructor the row's segment so the entries can read it, and D8 makes a new perch one table row.
Step 2, one told lookup: D4 adds `Env.Setting(module, key)` as a caller-filled closure, so `shedrecipe` still never learns the file format; D5 has rows name `module.key`; D6 refuses an unknown key when the recipe builds, which is what the entry asks for to replace the lost compile-time check.
The entry's standalone and test bound holds: the hub fills `Setting` from `configreg`, and `envkit` and the `shedrecipe` fixture fill their own.

The entry names `RowClusterFans` and `RowFixFans` as maps to fold into `Segments`, but D5 moves them behind `Setting` instead.
That is no deviation: those fields still leave `Env`, the entry's step 2 covers plain single-key config values, and D5 states the split.
Leaving run-wide scalars and batten's `Setting` fill out matches the entry, which names only the per-row maps and requires no batten row to read a setting.

Board fit:

- `row-entry-gate` plans to read its gates through this task's `Setting(module, key)` when a row is reached.
  D4's plain func type and its rule that a caller wanting a fresh read makes a new lookup leave that open, and the decision record lists the batten fill and the darn edge rewrite as that entry's work under Open risks.
  No duplication or contradiction.
- `command-producer` and `shed-arbiter-all` touch other rows and seams; neither is duplicated.
- `config-migrate-hub-layer` (#709) stays out of scope and is recorded as a residual risk.

Overlap: `lyx loom peers` lists no peer run in flight (`own_row` 3).
The one parent commit it lists, `2f330ee` (creel CLI output fix), changes only `internal/creelcli`, `internal/creelengine` and a sandbox doc, and overlaps this run nowhere, so no landing order applies.
