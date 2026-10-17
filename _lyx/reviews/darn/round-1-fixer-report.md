## Changes

All eight findings fixed as proposed, except F3 where the proposal's second option was taken (no code move).

- F1: added `TestRunner_Start_LedgerWriteFailure_CleansRunDirAndRegistersNoStrand` in `internal/shuttleengine/run_test.go`, with a directory at `engines-run.json`; it asserts the error, no `AddStrand` call and no leftover run directory.
  Raised the `internal/shuttleengine` row in `cmd/lyx/testdata/test-budget.yaml`.
- F2: `migrateCoAuthoredBy` drops an empty, whitespace-only or non-scalar `co_authored_by` without creating the claude entry; rows added to `TestMigrateConfig`; `MigrateConfig` doc comment and `internal/landingshed/doc.go` updated.
- F3: took the doc-comment option, because moving `recordEngineRun` after `AddStrand` would let a ledger failure leave a live strand and contradict F1's pinned behaviour; `recordEngineRun` now states the ledger counts start attempts that passed Prepare.
- F4: the `EnginesRun_counts_a_swept_run_and_a_record_without_an_engine` subtest runs the real `sweepOrphans` and asserts `engines-run.json` and its lock survive.
- F5: added a `NilEnginesRun` row to `TestFinalize_MergeOptionsCarriesComposedMessage`.
- F6: added `ExistingEntriesKept` and `TestMigrateConfig_InvalidDocument` (unparsable and non-mapping root, both `configengine.ErrInvalid`); raised the `internal/landingshed` budget row.
- F7: both describe stencils now say the landing step appends the configured trailer block itself.
- F8: removed the references to `CommitStatus`, `landingDeps` and `Start` from the `EnginesRun` and ledger doc comments.

## Disputed

None.

## Deferred

None.

## Self-check

- fix-consistency: no findings.
- fix-code-correctness: no findings.
- fix-affected-set, LOW: `internal/landingshed/doc.go:117` still described the migration without the empty-value drop; fixed as proposed.
- fix-proposal, LOW: same stale `doc.go:117`; merged with the above and fixed.
- fix-proposal, LOW: the `recordEngineRun` comment named only strand-registration failure; reworded to cover any later failure.
- fix-proposal, LOW: the F1 test did not tie the error to the ledger branch; it now requires "record engine run" in the error.
