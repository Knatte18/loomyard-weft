# Fixer report — Discussion-Review round 1

Every finding is fixed.
`lyx loom validate-discussion` exits 0 on the edited pair.

## Changes

- **F1 (D6):** added a scan on `scankit` as the membership oracle: every production package declaring `ConfigTemplate` is registered in `Modules()`.
  Added a Bound line stating that the property test no longer notices a dropped module and that the scan does.
  The `configreg` acceptance criterion gains a planted-removal proof.
- **F2 (D9):** added a Bound line.
  The property pins `MaxBounces` and `Segment` only as pair-equal, except Discussion-Bouncer's budget, which `resume_test.go` drives.
  `OnDone` order is guarded by `sequence_test.go`, and the PR-Gate and PR-Rework edges by `TestRecipe_PublishRoutesThroughPRGate`.
- **F3 (D14, D20):** added a Bound line to D14.
  Deletion is case by case: a `loomcli` case with no counterpart in `shedverbs` moves there first, with the three missing `PrintStatusLinesOnChange` cases named.
  `loomcli`'s own wiring stays asserted by the loom-specific tests D14 keeps.
  Removed D20's false claim about `step_test.go`.
  The D14 acceptance criterion now requires every deleted case to be asserted in `shedverbs`.
- **F4 (D18):** the allowlist starts empty.
  An entry is admitted only for a package whose `TestMain` cannot call the entry point, and its reason names that obstacle; "does not start tmux" is not a reason.
- **F5 (D19, Constraints):** no other kit imports `tmuxkit`, as for `lyxbin`.
  The rebuilt Testkit scan enforces this, the Bound line names it, the Constraints line adds it, and the `tmuxkit` acceptance criterion gains a planted-import proof.
- **F6 (Acceptance criteria):** a new socket is excluded only when its key equals `reedengine.ServerName` of one of the operator's own hub paths.
  Any other new socket fails the criterion, whether or not its server answers.
- **F7 (D20):** the list of invariants with unchanged text stays in D20.
  Moved the per-invariant justifications and the Rationale line to the support log's Interview item 10.
- **F8 (Rationale lines):** dropped the churn-statistic Rationale lines of D1, D4, D5, D6, D8, D9 and D12.
  D10's Rationale is reduced to the second-writer clause.
  The evidence moved to a new support-log Interview item 11.
- **F9 (Notes):** dropped the reed `-L` Note and folded `internal/reedengine` into D17's sentence.
  Dropped the `validate.go` Note, which restated D14.
- **F10 (D4):** reworded to say that the LookPath skips and the `go env GOMOD` execs are removed.

## Deferred

None.
