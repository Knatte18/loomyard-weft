---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/creelengine/server.go:411
    origin: lens:code-tests
    summary: no test drives awaitOutcome's ring branch through the daemon, so a ring answered then lost to a client close or daemon stop, or a failed ring write, has no row
    fix: In internal/creelengine/server_test.go add rows beside AMarkedHookWaitWhoseClientLeavesBeforeTheAck that, through a real dial, (a) queue a ring outcome in the waiter's channel before the connection closes or the daemon stops and assert the claim is released and the mail stays unread for a later claim, and (b) close the client between the ring write and the ack and assert the same release; order the queued outcome before the close with a small seam in export_test.go as the decision record's note for the plan writer describes.
  - id: F2
    severity: NIT
    class: consistency
    location: internal/creelengine/pool.go:51
    origin: handler
    summary: the stopBlocks field comment still says Stop blocks, though a Stop claim is now additional context that continues the turn and the neighbouring comments were reworded
    fix: In internal/creelengine/pool.go change the stopBlocks field comment to say it maps a strand to the Stop claims that handed out mail in its current turn, matching stopBlockCap's comment in verbs.go.
---

### [LOW] Ring answer-path ends are not tested through awaitOutcome

**Location:** internal/creelengine/server.go:411
**Issue:** D3 requires that every end of the ring's answer path before the journaled acknowledgement releases the claim, among them the client leaving or the daemon stopping after the ring was answered but before the outcome was written.
The release itself is correct in the code: the deferred `Waiter.Cancel` drains an unread ring outcome and releases its `Reading`, and `awaitAck` releases on a failed ack read.
The only rows reaching it are the store-level `ARingOutcomeNeverTakenIsReleasedWhenItsWaitIsCancelled` and the socket row for a client leaving before the ack, so the wiring of `awaitOutcome`'s select to `Cancel`, and the ignored failed write of the ring response, have no row that would fail if broken.

### [NIT] Stale Stop-block wording in a field comment

**Location:** internal/creelengine/pool.go:51
**Issue:** The `stopBlocks` field comment still reads "the Stop blocks of its current turn", while `stopBlockCap`, the package doc and the claudeengine doc now describe Stop claims that continue the turn.
The identifier stays as the plan keeps it; only the comment is stale.

## Rejected

- Lens code-tests, no test for an unmarked waiter rung without mail: rejected, `internal/creelengine/deliver_test.go:1855` asserts the unmarked waiter is answered with a ring carrying no claim and that a later claim hands out the mail.
- Lens code-tests, no test that an echo-only claim neither answers the ring nor counts a Stop: rejected, the lens read test names only; the echo-alone claim rows at `internal/creelengine/deliver_test.go:2050-2084` run the paused, held and Stop-cap claims that hand out an echo alone, and the code path (`ClaimAtBoundary` returns before the mark and count when no message is claimed) is unambiguous.
- Lens code-correctness, `typeBell` logs an Info line for every busy send: rejected, D5 asks for an Info line per bell send, `typeBell` returns before sending while the pane is not idle, and Info in a retry is not the Warn the log-level policy bars.
- Verbs_test.go:1261 "Stop block %d" test message: rejected, test wording only, no behavior or doc contradicted.
- Lens code-affected-set raised no finding.
