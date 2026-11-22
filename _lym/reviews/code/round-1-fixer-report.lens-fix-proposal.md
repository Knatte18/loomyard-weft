```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: challenge the proposals carried out as proposed (F1, F3, F4, F6, F7, F8).

No findings of BLOCKING or MEDIUM severity.

- **LOW: `internal/loomcli/drivewiring.go`, `handBackNotify` (F3).**
  - The extraction lets the test call `handBackNotify(pair)` directly. Nothing pins that `HandBack.Notify` is assigned `handBackNotify(location)`.
  - A revert of that one field to an unstamped `orchcli.NotifyParent` closure fails no test.
  - The review's seam was the `HandBack.Notify` wiring, which `goDriverDepsFor` builds. The converted closure is pinned, but its assignment is not.

- **LOW: `internal/loomcli/gotowiring_test.go` (F4).**
  - The test only asserts `Admitted != nil`. It does not call the hook with `loomshed.NamePRReview` and check that autonomy and strand are cleared, which the review's `fix` asked for.
  - Swapping `prReviewGotoAdmitted()` for any other non-nil function passes.
  - Nothing pins that `loomConsumeAttempt` and the `ConsumeTackler` closure in `goDriverDepsFor` call `relaunchRequest`. Putting an inline `RelaunchRequest` back in either place passes.
  - Evidence: `relaunchRequest("run-1", nil)` is the only caller in the test.

- **LOW: `internal/loomcli/pawldrain_integration_test.go` (F7).**
  - The member entry has no `status`, and the rows are named "open draft member".
  - A task with an empty status becomes `draft` only at the board write critical section (`boardengine/doc.go:199-200`). The test writes `board.json` directly with `writeConfigFile`, so the entry stays status-less.
  - `beamMembers` counts any non-`done` task, so the row passes. It pins "not done", not "draft counts". A rule that skipped `StatusDraft` would still pass.
  - Fix: give the member `"status":"draft"` in the JSON.

- **No defect found in the other proposals.**
  - F1: `ClearReviewSession` clears `ParentReviewAsked`, and its only caller is the goto reset.
  - F6: the `wantErrText` marks exemption applies only to the one new row.
  - F8: docs only.

Further tool calls:
1. `grep -i draft internal/boardengine/*.go`: to check how a draft member is represented, which my context lacked.
