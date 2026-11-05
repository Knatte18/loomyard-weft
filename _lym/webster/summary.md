# drift-rework: stuck at batch 20 on a plan conflict

Batches 01, 05, 09, 11, 13, 15, 16 and 17 are recorded `done`. Batch 20 and every batch after it did not begin.

## Batches done

- **01 (cards 1 to 4):** weft trailer commits, checkout states, the capture diff, and the new drift config keys.
  Deviations: files outside the card targets in `internal/driftcli`, `internal/driftengine` and `internal/fabricengine`.
- **05 (cards 5 to 8):** the expectation and judgement parsers, the records guard, and the seal, baseline-digest and row-order checks.
  Departure: `PATTERN-fabric-vocabulary`, with `TrailerCommit.WarpSHA` renamed to `CodeHead`.
  Deviations include `internal/testkit/envkit/envkit.go` and `internal/fabricengine/trailercommit.go`.
- **09 (cards 9 and 10):** slot before lock, the current capture, the shared check and the `drift` gate entry.
  Deviations include `internal/shedrecipe/entries_gate.go` and `internal/loomrecipe/gatequiescence_test.go`.
- **11 (cards 11 and 12):** run-level entry reports and the Drift-Expect producer.
  Deviations include `internal/shedrecipe/coverage_guard_test.go` and `internal/driftengine/seats.go`.
- **13 (cards 13 and 14):** the Drift-Expect rows and `lym drift check`.
  Deviations include `internal/driftcli/cli.go` and `internal/shedrecipe/coverage_guard_test.go`.
- **15 (card 15):** the judge seat, stencil and snapshot guard.
  Departure: `PATTERN-fabric-vocabulary`, with `Deps.WeftChangedSince` renamed to `RecordsChangedSince`, `Deps.LoomWeftDirs` to `LoomRecordsDirs`, and `fabricengine.WeftPathsChangedSince` to `RecordsPathsChangedSince`.
- **16 (card 16):** Drift-Verify as containment and judge under lines mode, and `Deps.EndRoles`.
  Deviations include `internal/loomcli/driftwiring.go` and `internal/shedrecipe/coverage_guard_test.go`.
- **17 (cards 17 to 19):** the command mode, the code-only Drift-Baseline, Drift-Await's digest, `lym drift amend` and the wider `lym drift defer`.
  Deviations include `internal/driftengine/deps.go` and `internal/loomcli/driftwiring_integration_test.go`.

## Why the run stopped

`begin-batch 20` refused with `plan_conflict: tree_moved`.
The finding: card 22 deletes `internal/driftcli#compareLockRetry`, but card 26 still references it at `internal/driftcli/wiring.go:247` in its Edit code, so the delete must move to a card after card 26.
`lym webster fix-plan` returned verdict `plan_writer` and edited no card.
The plan needs a plan writer to reorder the delete, then `lym webster rebaseline --card NN` for each edited card, and the run resumes with `lym webster run`.
