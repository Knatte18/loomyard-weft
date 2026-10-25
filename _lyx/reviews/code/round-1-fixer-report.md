# Fixer-report, code round 1

## Changes

All four findings were fixed as proposed, one commit each.

- **F1** (`05b117605`): `RunCompare` takes the compare marker with `lock.AcquireWriteLockWithin` and the new `CompareRequest.LockRetry`, and refuses only when the marker is still held after the retry.
  Both production callers (`internal/driftcli/cli.go`, `internal/loomcli/driftwiring.go`) pass a 2s bound through their own constants.
  The `TestRunCompare` held-marker row now covers a marker held past `LockRetry` (refuses) and one held briefly inside it (the compare runs); the fixture sets `LockRetry`.
  `internal/driftengine/doc.go` states the bounded retry.
- **F2** (`d41205f04`): the ruling const group and `Acceptance.Ruling` comments name scope full, and `RulingUnruled` and `RulingRuled` name the Drift-Verify agent session and the run's parent.
- **F3** (`e29d3558a`): `EndForCancel` calls `t.killCapture(req.Slug)` after the caller's hook and before the strand stop, for a present and a gone task worktree, and returns its error wrapped as `EndSession` does, with the cancel's retry command.
  `TestEndForCancel` rows carry the `killCapture` call, and a new row asserts a failed termination stops the cancel before any strand is stopped.
  `internal/pairteardown/doc.go` gains the step and is renumbered.
- **F4** (`babbc8b65`): `CaptureRequest.Check` now returns the HEAD it verified under the lock, `RunCapture` records that value, and `CaptureRequest.Head` is removed.
  `checkMainCapture` in `internal/driftcli/cli.go` and `checkMain` in `internal/loomcli/driftwiring.go` return the checked HEAD; the earlier reads before the lock are gone.
  The existing `rec.Head` assertion in `TestRunCapture` now exercises the value `Check` returns.
- **Self-check follow-ups** (last commit): `pattern/PATTERN-pair-teardown.md` names the capture job termination in the cancel phase, and `CompareRequest.LockRetry`'s comment says it must be positive.

Verification: `lyx gate test --tier 1` over `internal/driftengine`, `internal/driftcli` and `internal/loomcli` passed, and `--tier 0` over `internal/pairteardown`, `internal/battencli`, `internal/driftengine`, `internal/driftcli` and `internal/loomcli` passed.
The final two follow-up edits are a comment and a markdown line and were not re-run.

## Disputed

None.

## Deferred

None.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW: `pattern/PATTERN-pair-teardown.md` omitted the capture job termination; fixed as proposed.
- fix-proposal, LOW: a zero `CompareRequest.LockRetry` refuses even a free marker; fixed as proposed by documenting it in the field comment.
  The same trap on `CaptureRequest.LockRetry` predates this round and was left as it was.
- fix-affected-set: no findings.
