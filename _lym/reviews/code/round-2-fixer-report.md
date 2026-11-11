# Fixer report — code round 2

## Changes

- **F1** fixed as proposed.
  `internal/loomcli/decision_test.go` gains the row `decision lock cannot be taken` in `TestDecisionVerb_Refusals`, driving `decisionVerb` into the lock failure.
  It wants "decision lock", "way forward" and "re-run the verb; nothing was written", and the table's shared checks assert no commit, no append, no post and an untouched record.
  Commit: `loomcli: fix F1`.
- **F2** fixed as proposed.
  `decisionFake` gains `writeFirstErr` and `writeAllErr`, honoured in its `writeRecord` closure.
  `TestDecisionReplaceVerb_Refusals` gains `WriteFailsAndTheRestoreSucceeds` and `WriteAndRestoreBothFail`.
  The fake's fields and the `cmp` import landed in the F1 commit, since they share a file; the replace rows are in the F2 commit.
- **F3** fixed as proposed.
  `internal/boardengine/runlock.go` and `internal/boardengine/doc.go` name both `lym loom decision add` and `lym loom decision replace` as the ways the record changes.
  Commit: `boardengine: fix F3`.
- Self-check follow-up: the reworded `doc.go` line ran past 120 characters, so it is split at the semicolon.

`lym gate test ./internal/loomcli` passed before the self-check.

## Self-check

- fix-code-correctness, LOW: the `doc.go` line over 120 characters. Fixed as proposed.
- fix-proposal, LOW: the same `doc.go` line, a duplicate. Fixed as proposed.
- fix-consistency: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
