# Fixer-report, code review round 1

## Changes

- **F2** (fixed as proposed), commit `fe93350e`: the `stopBlocks` field comment in `internal/creelengine/pool.go` now says it maps a strand to the Stop claims that handed out mail in its current turn.
- **F1** (fixed differently), commits `e4779364` and `11febd4d`:
  - Added the test seam `Options.beforeOutcomeWait` in `internal/creelengine/server.go`, called in `awaitOutcome` after the close watch starts and ahead of the select, with the waiter and the close watch's channel.
    It is nil in production.
  - `WithBeforeOutcomeWait` in `internal/creelengine/export_test.go` hands a hook two predicates: an outcome stands unread in the waiter's channel, and the close watch saw the client leave.
  - New row `AMarkedHookWaitWhoseClientLeavesAfterTheRingWasQueuedLeavesTheMailUnread` in `internal/creelengine/server_test.go`: for several rounds the hook holds the wait until the ring is queued, closes the client and waits for the watch to see it, so the select sees both ready.
    Each round asserts a later `read --wait` returns the mail.
    Run with the drain in `Waiter.Cancel` removed, the row fails, so it pins the wiring of `awaitOutcome` to `Cancel`.
  - Part (b) of the proposed fix, closing the client between the ring write and the ack, is not added: the existing row `AMarkedHookWaitWhoseClientLeavesBeforeTheAckLeavesTheMailUnread` already drives it, and a second row would be redundant.
  - The shared dial code of `dialHookWait` is split out as `sendHookWait` so the row can open one connection per round.

## Self-check

- fix-consistency: no findings.
- fix-code-correctness, MEDIUM: the first version of the row did not reliably reach the `Cancel` drain, because the hook returned before the close watch saw the client leave. Fixed in `11febd4d` with the clientLeft predicate and repeated rounds.
- fix-proposal, LOW: same defect as the correctness finding, merged with it and fixed by the same change.
- fix-proposal, LOW: the daemon-stop variant has no row. Deferred, see below.
- fix-affected-set, LOW: the daemon-stop end and the failed ring write are not pinned. The failed ring write is reached whenever the select picks the ring branch after the client left, which the repeated rounds do; the daemon-stop end is deferred, see below.
- Lens reports are saved beside this report as `round-1-fixer-report.lens-<lens>.md`.

## Disputed

None.

## Deferred

- F1, the daemon-stop end (`<-d.done`) of the ring's answer path.
  The hook cannot await the shutdown, so the ring and the stop cannot be ordered the way the close is, and the release shows only after a restart on the same journal.
  A fixer alone cannot settle how to order that stop without a second seam in `Run`, which is a design choice for the plan writer.
