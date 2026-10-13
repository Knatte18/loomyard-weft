# Plan round 1 — fixer report

## Changes

- **F1** — fixed as proposed, plus two test-plumbing additions its new rows need.
  `00-overview.md`: the paragraph above the D11 table now says the audit swept every loop and polled path, and drops the claim that a line outside named loops needs no verdict.
  The table gains guard rows for `Run.handleGatedBoundary`'s `pending send failed` and `Run.checkLivenessTick`'s `dismiss trust prompt (non-fatal)`, a keep row for `persist run started failed`, and keep rows for the `reedengine` boot loop, `seatengine` `noticeSender.deliver`, the `fabricengine` push loop and `landingshed` `PRAlign.wait`.
  `10-shuttle-read-guards.md`: `TestGatePending_FailedSendWarnsAndStaysPending` is under **Edit:**, the opening sentence names the sites the table lists for the package, the Intent adds the `pending:<entry>` and `startup dismiss` episodes with their recoveries, and the Tests add the three-pending-evaluation Debug assertion and the failing-dismissal row.
  The failing-dismissal row of `TestRun_Wait_StartupWindow` is driven through the fake's existing `SendKeyErr`, so the card adds `captureErrQueue` and `sendKeyErr` columns to that test (the review's fix did not name how the row gets its failure).
- **F2** — fixed as proposed, with one fixed-differently detail.
  Card 3's `UnreadableClientListStillRenames` row runs a third, readable repair and asserts the Info recovery line.
  Card 10's expiry row errors on its first two re-checks through a new test-local `probeErrCalls` column, then asserts one WARN and one `expiry probe answers again` INFO.
  Card 10's startup-capture row uses the fake's existing `CaptureErrQueue`.
  Card 11's network-marker row replaces the plain file with an empty directory, ticks once more, and asserts the recovery line.
  Fixed differently: the review said to add no new target, but the activity guard's recovery cannot be reached with the fake as it stands, since `fakeReed.OperatorActive` fails every call while `ActivityErr` is set (`internal/shuttleengine/fakes_test.go:158`).
  Card 10 therefore lists `fakeReed` and `fakeReed.OperatorActive` under **Edit:** and gives the fake an `ActivityErrQueue []error`, mirroring `CaptureErrQueue`.
  The new `TestSend_WaitsForIdleSession` row drives that queue.
  No new test function was added, so no test-budget row changes.

`lyx loom validate-plan` reports `ok: true` after the edits.

## Disputed

None.

## Deferred

None.
