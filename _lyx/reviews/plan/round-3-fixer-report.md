# Plan round 3 — fixer report

Every finding was checked against the code and the decision record, and every one holds; all eight are fixed.
`lyx loom validate-plan` passes over the result, with informational findings only.

## Fixed

- **F1 (test budget).**
  Counted every package's top-level tests against `cmd/lyx/testdata/test-budget.yaml`; the packages named sit at their row.
  Fixed as a method: each card that adds a test now names it as a new top-level test, and then lists `cmd/lyx/testdata/test-budget.yaml` and states the raise, or names it as a row or step in an existing test.
  New tests with a raise: card 3 `TestMigrateConfig` (batcher), card 5 `TestLoomPreflight_Call_BatcherConfig` (loomshed), card 8 `TestMerriamBase`, card 10 `TestResetKeep` (gitrepo), card 11 `TestPlanReset_Resolution` (`reset_test.go` held no table, though card 11 said "gains rows"), card 12 `TestArchiveRunAfterReset`, card 14 `TestKeepResetRefusal`, card 16 the scan and its self-test (cmd/lyx +2), card 17 `TestRecoverBatch_UncommittedPathsPrompt`, card 19 `TestRecoverBatch_AmendedCards`, card 20 `TestOverviewWithoutCardIndex` (planparser), card 22 `TestPlanCommitPathspec` (loomcli).
  Rows or steps in named existing tests: card 13's and card 14's PlanReset rows in card 11's table, card 17's runwrites step, card 18's state and CLI steps, card 19's `TestFailBatch`, `TestRecordBatch_AuditOutcomes` and envelope steps, card 21's fingerprint rows.
- **F2.** Card 10 adds `cmd/lyx/gitrepoboundary_test.go`, whose `gitrepoPinnedRunBoundMethods` gains `ResetKeep`.
- **F3.** Card 22 adds `internal/shedrecipe/fixture_test.go`: `newTestEnv` fills `Rework.ArchiveWebster`, which keeps the `conformance_test.go` PlanWrite row, `gateCapableEntries` and the planwrite entry tests constructing.
- **F4.** Card 12 gives `headBeforeEveryStart` the caller's re-run steps beside `freshPendingDrop`'s.
  The reset caller's step for it and for HEAD not at the start is `lyx webster accept-audit` (or `--batch NN`), and the suspect-path step reuses `restoreStep`.
  Card 16 now rewords only the `Run` caller's texts of both functions.
- **F5.** Card 12 is split into three cards, and old cards 13–20 are renumbered 15–22, with every cross-reference, the Card Index and the Shared Decisions' `Applies to` lines updated:
  12 `run-archive-after-reset` (the factored archive and its guard, exported as `ArchiveRunAfterReset`; `Run --fresh` keeps its order and behavior), 13 `reset-start-archives` (pair reset to start archives, the archive-only `PlanReset` path), 14 `standalone-reset` (keep-reset in webstercli, `KeepResetRefusal`, the `Standalone` flag).
  Each refusal-spec row lands in the card that introduces or changes that refusal.
- **F6.** Card 16 adds `websterCLI.runCmd`, whose mismatch sentence now says the refusal names its way forward.
  Card 13 rewrites `resetCmd`'s "changes no other webster state" sentence and its refusal list for `start`, and card 14 rewrites the standalone sentence.
- **F7.** Card 12's new archive test covers only the guard refusals with the reset caller's way forward and the record left in place.
  The rename and prompt clearing stay with `TestArchiveRunFile`, `TestArchiveReportsDir` and `TestRun_EntryHousekeeping`.
- **F8.** Card 13 creates `websterengine.UncommittedPaths(geom)`: `DirtyPaths` minus `runOwnPath`.
  Accept-audit's dirt check in `AcceptBatchFabricReference` now calls it, and cards 13 and 14 use it for the `uncommitted` key.

## Gate repair after the fix

The round's gate then reported two blocking classes on the split, both fixed:
- `containment-file-overlap` (cards 13 and 14 against card 11): card 11 listed the whole `internal/websterengine/reset.go` while cards 13 and 14 list members of it.
  Card 11 now lists the members it changes, `ResetDeps`, `PlanReset` and `resolveResetSHA`, with any new helper sitting beside `resolveResetSHA`.
- `handle-unreferenced` (card 14): no other card referenced `plan:internal/websterengine#KeepResetRefusal`.
  Card 14 now creates the file `internal/websterengine/keepreset.go`, holding `KeepResetRefusal` as its one function, instead of declaring a handle; its test stays in `reset_test.go`.
`lyx loom validate-plan` passes after the repair, with informational findings only.

## Disputed

None.

## Deferred

None.
