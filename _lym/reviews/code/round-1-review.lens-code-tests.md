```yaml
lens: code-tests
outcome: reported
```

**Per-behaviour list (code-tests lens)**

1. **A `decision` escalation is eligible, and `budget` stays refused.** Pinned by `TestEscalationEligible`, rows "pending decision" (`want: true`) and "pending budget", at `internal/arbiterstop/arbiterstop_test.go:136-139`. It runs through `EscalationEligible`, `Check` and `Eligibility`, and the refused rows also run `Check` and want the `HandBackWayForward` suffix. If the new `case` clause in `arbiterstop.go:107` were dropped, the "pending decision" row would fail.

2. **An arbiter `resume` at an awaiting stop with an orch recorded sends exactly one notice, parks nothing, writes no hand-back report, and relays no parent notice.** Pinned by the new `TestGoDriver_Settle` row at `drive_test.go:452-458` (`wantNotifies: 1`, `wantReports: 1`, `wantParked` false, `wantExit: ExitDone`). The count would fail if the call were removed or doubled. A relayed `ParentNotice` would also push the count to 2.

3. **The notice text names the run, the stop's producer and the verdict path, and ends "Nothing to answer."** None. The harness at `drive_test.go:750` and `:947-948` only counts `Notify` calls and never checks the line. Changing or emptying the line fails no row.

4. **The notice is sent only at an awaiting stop (`stop.Key().State == awaiting` guard at `drive.go:554`).** None. The other rows where the arbiter resumes at a blocked stop, `drive_test.go:272` and `:349`, leave `orchRecorded` false, so the guard-less code would still give zero `Notify` calls there. The only blocked-stop rows with `orchRecorded: true` (`:405`, `:412`) hand on and never reach the resume branch.

5. **With no orch strand recorded, the line is logged and nothing is posted (`OrchRecorded` false).** None. No row has an arbiter `resume` at an awaiting stop with `orchRecorded` false. The `OrchRecorded` stub at `drive_test.go:755` returns the row's field, but only the hand-on rows run the false case for the hand-back path, not `noticeArbiterSettled`.

6. **An `OrchRecorded` error is logged, the notice is not sent, and the driver goes on.** None. The stub at `drive_test.go:755` never returns an error.

7. **A failed `Notify` send is logged and does not stop the driver.** None. The `Notify` stub at `drive_test.go:750` is not shown returning an error for any row in what I read. Nothing pins that a send failure still leaves the run resuming, as the doc comment on `noticeArbiterSettled` claims.

8. **`Check` returns the brief path for an admitted `decision` escalation.** None specific to `decision`. The `TestEscalationEligible` fakes return `"brief.md"`, but `EscalationEligible` discards the path, and no row asserts `Check`'s returned `briefPath` for any cause. This predates the diff; I note it only because the focus file asks for `Check` coverage of the eligibility change.

**Findings**

- **MEDIUM, `design`, `internal/sheddriver/drive_test.go:452-458`.** The new behaviour in `noticeArbiterSettled` is only partly covered. Items 5 and 6 have no row, and neither has item 7's failed-send path. Item 3's line content has no row either. Evidence: `drive.go:565-577` has three branches, and the test row exercises only the success branch, counted by `drive_test.go:947`.
- **LOW, `design`, `drive_test.go:272, 349`.** The awaiting-only guard (item 4) is unpinned: no row has an arbiter `resume` at a blocked stop with `orchRecorded: true`. Removing the `State == awaiting` check at `drive.go:554` would break no test.
- **NIT, `design`, `drive_test.go:947`.** Item 3: the harness counts `Notify` calls but never records the line. The fix is a `wantNotice` substring assertion on the one new row.

**Tool calls: 1.** A `grep` of `drive_test.go` for the `arbiterOutcome("resume"` rows, their `orchRecorded` values and the `Notify`/`wantNotifies` assertions. My context had the new row and a few neighbours, but not every resume row's `orchRecorded` setting or what the harness asserts about the line.
