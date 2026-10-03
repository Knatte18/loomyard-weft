# Fixer report — plan, round 2

All five findings are fixed in the plan cards.

- **F1 (BLOCKING):** card 10 now edits `internal/gitrepo/noforceadd_test.go`.
  Its intent converts the Never Force-Add scan to a shallow production-file `scankit` walk of its own package directory, keeping `noForceAddBannedTokens` and the floor of 5 verbatim, and proves it with one planted `"-f"`.
- **F2 (MEDIUM):** card 15 now edits `internal/loomrecipe/names.go`.
  `RecipeEngines`' doc comment names the one row table in place of `loomRowEngines`, and the ImpactSummary records the one production doc-comment edit.
- **F3 (MEDIUM):** card 11 now edits `cmd/lyx/seamsignature_test.go` and `cmd/lyx/sandbox_coverage_test.go`.
  Their comments name `clitree_test.go`'s walk in place of the deleted `drift_test.go` and `longlist_test.go`, and the ImpactSummary covers the change.
- **F4 (MEDIUM):** card 28 now states that `loomcli` keeps its `commitStatusDeps` type with its flat field set and its `newCommitStatusSeam(deps)` signature, as a wrapper over the shared core.
  It names `commitrecords.go`, `commitrecords_test.go` and the protected integration test as the reason, and says only the message renderer, the failure disposition and the core closure move.
  The ImpactSummary matches.
- **F5 (LOW):** card 27's intent now has `shedverbs` test comments that cite a deleted `loomcli` test rewritten to stand alone, with `TestStepEnvelope_KeySetIsExactlySixteen`'s comment as the example.

The `approved:` key in `00-overview.md` is untouched.

## Deferred

None.
