# Fixer-report, round 1

## Changes

- F1 (BLOCKING), fixed as proposed: `internal/reedcli/watchdog.go` declares `var consumeFailure logger.Episode` beside `unwatched`.
  The unwatched timer arm logs a consume error through `consumeFailure.Fail` and calls `consumeFailure.Recover` on a successful consume.
  The events-branch Warn is unchanged.
- F2 (LOW), fixed as proposed: `internal/reedcli/watchdog_test.go` gains a `closedWatch` field and the row "stop with an idle listing and a closed watch".
  Its opener returns an already-closed channel, and it asserts the closed-watch trace line and "exiting on a stop request".
  The test's doc comment now names `closedWatch` beside `failWatch`, as the self-check found it stale.
- F3 (LOW), fixed as proposed: the comment at `internal/reedengine/naming_integration_test.go:69` says spawn skips allow-set-title on a tmux older than 3.5.

`lyx gate test ./internal/reedcli` passed after F1 and F2.
The final doc-comment edit changes no code.

## Self-check

- fix-consistency, LOW: the test doc comment was stale for the `closedWatch` row; fixed as proposed.
- fix-code-correctness, LOW: the same stale doc comment, a duplicate; fixed as proposed.
- fix-affected-set: no findings.
- fix-proposal: no findings.

## Disputed

None.

## Deferred

None.
