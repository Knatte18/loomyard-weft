---
verdict: APPROVED
---

## Scope check

The decision record covers every point the board entry `dotnet-scoped-verify` keeps in scope, and nothing beyond it.

- Map changed code to the tests that exercise it and run them through `dotnet test --filter`: Decisions 1 to 4, a name closure selecting test classes inside shared test projects, as the entry's note on shared test projects asks.
- Restore once, then build and test incrementally: Decision 6, met by the build step's implicit no-op restore and the tiers' `--no-build` runs.
- Keep build processes warm: Decision 7.
- The whole solution once, at the final gate: Decision 10, with Merge-In as the one whole run and Finalize carrying.
- How a missed test is caught: by the whole gates (Decision 2, Q2), one of the two options the entry lists.
- Measuring before and after on the work machine: left to the operator with `lyx gate report`, with the docs telling them to take the baseline first, since Models is never cloned outside OSL-1046 (note `csharp-sandbox-hub`).
- Wanted 4: card-scoped in-batch checks, a whole pass of every tier (the slow tier included) carried at a later gate on the same tree, and doc-only diffs skipping every tier through `inert_paths`, a hub-declared generalisation of the entry's `*.md` rule.
- Wanted 5: a `slots: all`, `manual: true`, `carry: false` tier for timing work, with correctness runs still sharing slots.
- Wanted 6: one shared build step whose command carries the gate properties, then `--no-build` tiers.
- Repo rules: lens prose in the hub's `lenses.yaml`, shown in `docs/dotnet-hub.md`, with no rule engine, as the entry says these are not tool features.
- Required 1 and 3 (task `test-tiers`) and Required 2 with its detached capture (note `regression-baseline-verify`) stay out of scope, as the entry assigns them.

No other entry's work is taken.
Note `regression-baseline-verify` relies on `carry: false` as a generic tier key and keeps its capture, compare and shed rows; Q11 follows that note's operator decision of 2026-10-21 that the toolchain lives in the gate config.
Note `csharp-sandbox-hub` keeps its hub config and first end-to-end run, which this design leaves to it.

## Overlap with peers and the parent

This run's `own_row` is 3.

- `run-profiles-ladder` (row 16): both change `docs/overview.md`, at different places; a text-only overlap with no design conflict. By the rule `run-profiles-ladder` lands first and this run takes its changes at Merge-In.
- `umbrella-groundwork` (row 6): its record adds the `beam` module to `docs/overview.md`'s module table, while this run edits the same file elsewhere; a bookkeeping overlap only. `umbrella-groundwork` lands first.
- `pattern-restore-caller-rules` (row 14): it adds `PATTERN.md` overview entries and rules background files out of its scope; this run changes no PATTERN entry and narrows only a line in the `pattern/` gate-slot background. No shared file, no overlap.
- `runs-never-write-board` (row 0, unknown order): its record touches no gate, scoper or docs file of this run. No overlap.
- Parent commit `c9c4e7d` (burlerengine: let forks run git in repositories they created): touches `internal/burlerengine` only. No overlap.
