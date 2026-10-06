# Round 2 fixer report

All seven findings fixed in `_lyx/discussion/decision-record.md`.
`support-log.md` is unchanged: the gate appends its own round entries.

## Changed

- F1: Decision 10's reason is now one of three: what the test pins, the core or testkit file that names it (Decision 1), or the assertion it received (Decision 2).
  "Never annotated" applies to a candidate with none of them.
  Decision 1's bound points at the new reason.
- F2: Decision 12 defines the figure as the covered fraction of the package's own non-test statements, from the profile of its own tests under `-coverpkg=./...` filtered by directory.
  Any drop is a mistake and is restored, except a block that flips between two runs of the unchanged tree, which the PR records as noise.
  The rationale and the coverage acceptance criterion now agree.
- F3: Decision 2 says a receiving test is never deleted later in the prune, and carries a keep reason if a rerun lists it.
- F4: Decision 9 separates the per-package reruns of Decision 10 from the final once-per-package run, puts scratch files under `.scratch/`, names the final card as the splice performer, and says a package left with no tests loses its section.
  "Adds or removes none" became "adds none".
- F5: Decision 5 adds a one-line comment above a test whose move is blocked by a missing shared fake, and keeps the PR body line as the hub hand-off.
- F6: The deleted-candidate acceptance criterion now requires the PR to list the receiving test.
- F7: Decision 8's command carries `<package>`.

## Deferred

None.
