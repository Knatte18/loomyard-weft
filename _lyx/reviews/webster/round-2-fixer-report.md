# Round 2 fixer report

All three findings of `round-2-review.md` are fixed, one commit each.
After the last commit `go build ./...` and `go test ./...` pass, as does `go test -tags integration` over `internal/websterengine`, `internal/webstercli` and `tools/tokencount`.

## Fixed

- **F1 (MEDIUM, scope) — commit `5c3570c4c`.**
  `internal/websterengine/runlevel_test.go` only; production code is unchanged.
  `TestRun_RefusesBeforeSpawn` gained a `fresh` field and a row, "a --fresh re-init whose batches run a dependency backward archives nothing".
  The row seeds a stale state, runs `--fresh` over a plan whose card 1 uses card 2's target, and asserts `ErrBatchOrder`, the way forward, the stale `state.json` still in place and no `state-*.json` archive.
  The `--fresh` case of `TestRun_EntryHousekeeping` now also asserts the re-initialised state records the active batchifier's partition.
  Both assertions were seen failing against a temporary mutation of `Run` (dropping `RecordPartition` from the `--fresh` branch, and moving `newPartition()` after `archiveStateFile`), and the production file was restored afterwards.
- **F2 (LOW, consistency) — commit `3688c966c`.**
  `internal/websterengine/rebaseline_test.go`: the `Rebaseline` half of the #330 test is removed, since its `ErrBatchOrder` expectation duplicates the `TestRebaseline_CardSet` row and its no-card-set check could not fail.
  The test is renamed `TestBeginBatch_RewrittenBegunCardMovesRecordedHash_Regression330` and its doc comment says what it pins and where the accept after a moved hash is pinned (`TestRebaseline_AfterRecordBatchBoundBegunCard_Regression330`).
  No test was added.
- **F3 (NIT, consistency) — commit `f1ef02627`.**
  `internal/batcher/estimate_test.go`: doc comments on `editCard`, `TestSegmentCost` and `TestDiskSizes`.

## Deferred

Nothing.

## Notes

No constraint conflicted with anything else in the prompt.
No friction note written.
