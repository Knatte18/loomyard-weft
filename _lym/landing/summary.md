# Retry a failed pr-review gate on pr-continue; stop cycling the session

A pr-review gate that failed for a reason outside the change, such as a file another process held open, deadlocked the row.
`gateHead` mails the failure once and records the head as `mailed_head`, so `gateDue` never gates that head again.
With the continue flag set, `judge` kept waiting for a gated head that would never come.
`sessionStopWanted` stopped and resumed the PR-Review session on every wake, about once a minute, without end.
The only way out was an empty commit that moved the head.

Closes #125

## What changes

- `lym loom pr-continue` now clears `MailedHead` along with setting `Continue`.
  The next wake sees a head the record has not gated or mailed, and gates it again.
  A rerun on a record that already carries `Continue` and a mailed head is written, not skipped as a no-op.
  A record with `ReviewEnded`, or with `Continue` and no mailed head, is still left untouched.
- The gate-failure mail now ends with a hint naming `lym loom pr-continue` as the way to gate the same head again (`gateRetryHint`).
  The PR-Review stencil gains a step with the same instruction and says to fix failures in the change itself in a new commit.
- `sessionStopWanted` takes the wake's head and dirty paths.
  It stops the session only when the read can end the review:
  a merged pull request, a recorded ending, the continue flag on a clean tree at the gated head, or an approval of the gated head.
  The continue flag on a dirty tree or an ungated head leaves the session running while the row waits.
- `wake` re-reads the tree only after a stop, and otherwise judges the tree it already read.

## Design decisions

- The retry is explicit: a failed head is still mailed once per head and never retried on its own, so a genuine failure in the change does not loop.
  Only the agent's `pr-continue` re-arms the gate.
- The mail hint is conditional on the operator having asked the agent to continue, so an agent does not run the verb to dodge a real failure.
- No new verb; `pr-continue` is reused as the explicit re-gate, as the issue allows.
- `docs/overview.md`, and the `landingshed`, `loomcli` and `pullrequest` doc comments describe the new behavior.

## To check

- Code review left one finding open: the session-stop cases in `internal/landingshed/prreview_wait_test.go` are not all pinned by the wait test.
  The recorded-ending and clean-gated-head stops are pinned; check that no stop case in `sessionStopWanted` is left uncovered.
- The change was not run against a live gate failure with a real PR-Review session.
  Run `lym loom pr-continue` after a mailed failure and confirm the row gates the same head once.
