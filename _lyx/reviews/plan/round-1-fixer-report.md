# Fixer report — plan, round 1

## Changes

- F1, fixed as proposed: card 2 drops the Edit targets `internal/batcher#TestIdentityBatcher_Batch` and `internal/websterengine_test#TestExecutionBatches` and the Uses entries `internal/websterengine_test#partitionPlan` and `internal/websterengine_test#batchIDs`.
  The two paragraphs that planned the redundant rows are replaced by one sentence naming the existing `threeCards` row and the recorded-partition row that keep covering both paths.
  Premise checked: `TestIdentityBatcher_Batch` holds the `threeCards` row (`internal/batcher/identity_test.go:24`), and `TestExecutionBatches` holds the row "a recorded partition is kept when the size source changes between calls" (`internal/websterengine/partition_test.go:119`).
- F2, fixed as proposed: card 6's `TestRecoverSpawnOrAttach` unreadable row and card 5's `TestScanGuardTests` unreadable row now name a root `go.mod`.
- F3, fixed as proposed, with the recommended option: card 5's Intent has the `scanGuardTests` walk open and close each collected `_test.go` file and return a failed open as the walk read error, which leaves `impactset`'s classification and the round gate's error policy unchanged.
  `TestScanGuardTests` gains a row with a root `go.mod` and a `0o000` `_test.go` file, giving an error naming that file and no `scanErr`.
  Premise checked: `guardNamesInFile` passes nil source to `parser.ParseFile` and wraps any failure as a `GuardScanError` (`internal/impactset/guards.go:69`).

`lyx loom validate-plan` reports `ok: true`, with only the informational create-new-unit findings.

## Disputed

None.

## Deferred

None.
