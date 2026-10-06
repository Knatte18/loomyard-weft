# Round 4 fixer report

All seven findings fixed; no git commands run.

## Changed

`_lyx/discussion/decision-record.md`:

- F1: Decision 13's eligibility bullet now requires independent tests, each step read-only on the fixture or restoring it, and drops the "comment names the change" escape.
  A test needing an earlier step's state is not merged.
- F2: Decision 13 gains a bullet that steps run serially, the scenario's top-level test calls `t.Parallel`, and a comment names the shared fixture as the reason.
- F3: the per-family timing command now carries `-count=1` and a package, both medians are taken back to back with the load average recorded, and a merge needs the after median to beat the before median by more than the before runs' spread.
- F4: Decision 13 states that a `failed` entry gets its Decision 8 verdict under its base name before it may be merged, and that a merged family's alone-rerun is the scenario name with `-count=1`.
- F5: Decision 12 now sums the profile's statement field, starts the run at the repository root, and ties a noise claim to a second run of the base commit.
- F6: Decision 2 lists merging among the paths for a `Removable: no` candidate, and Decision 12's rationale names merges.
- F7: removed the operator-provenance sentence from Decision 13.

`_lyx/discussion/support-log.md`:

- F7: removed the `_No rounds yet._` placeholder above the recorded round, and added Q12 to the Question ledger for Decision 13 with its provenance.

## Deferred

None.

## Notes

The writer stencil seeds `_No rounds yet._`.
The placeholder was removed because an entry now exists; if the gate expects it, the loop owner restores it.
