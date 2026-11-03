---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lym/landing/summary.md:27
    summary: the change description still says every failed rule read reads as no rule, but the code halts when a pull request is recorded
    fix: In _lym/landing/summary.md, change the bullet at line 27 and the design decision at lines 40-43 to say that a cancelled context and a transient failure are errors, that any other failure except an unusable token is a Halt when the run holds a pull request record, and that otherwise (no record, or an unusable token) a Warn is logged and the read counts as no rule. In the Tests section (lines 53-58), add the halt-with-a-record rows, the token soft-fail rows, the cancelled-context row and the empty-task and unusable-origin skip rows.
---

### [LOW] The change description contradicts the rule-read behavior the diff now has

**Location:** `_lym/landing/summary.md:27`, `_lym/landing/summary.md:40`
**Issue:** The summary says "Any other failure, an unusable token included, logs a Warn and reads as no rule" and "An unreadable rule falls back to `beam.yaml`".
`resolveWithParentRule` in `internal/landingshed/pullrequestmode.go:123` returns a Halt instead when `recordFound` is true and the failure is neither transient nor a token failure, and `internal/landingshed/doc.go` and `finalize.go` say the same.
The summary's Tests section also omits the rows the last two commits added.
The code is right and the claim is stale, so it needs correcting rather than the code.

## Focus answers

- **Rule read failing with a record (`resolveWithParentRule`, `Finalize`).**
  Confirmed.
  A non-transient, non-token failure with `recordFound` returns `PullRequestDecision{Halt: ...}`, and `pullRequestLanding` halts on it before `dropRecordedPullRequest` can run, so nothing is closed and the record stays.
  The soft Warn path stays only for a run with no record and for a token failure.
  A token failure with a record then reaches `dropRecordedPullRequest`, whose `API` and `Status` calls halt on the same token, so no direct push follows.
  Finalize's `AnUnreadableParentRuleWithARecordedPullRequestHaltsClosesNothingAndKeepsTheRecord` row pins this, with `wantRecordKept`.
- **Open pull request with `ReviewEnded` (`judge`, `openPullRequest`).**
  Confirmed.
  The `StateOpen` case returns Done on a clean tree and Stuck with `dirtyEndReason` naming the paths on a dirty one, as the `StateClosed` case does.
  Rows `ReviewEndedOnAnOpenPullRequestOnACleanTreeIsDoneWithoutEditingIt` and `ReviewEndedOnAnOpenPullRequestOnADirtyTreeHaltsNamingTheFiles` cover both.
- **Test rows for the added behaviors.**
  Each is pinned with exact call lists.
  Token soft-fail has rows in `prreview_test.go` and `finalize_pr_test.go`, and its error is a wrapped token error that the transient classifier reads as transient, so the rows fail without the `tokenFailure` guard.
  The cancelled-context error has the `ContextCancelledBeforeTheParentRuleReadIsAnErrorWrappingTheCancel` subtest, whose plain non-transient read error would soft-fail to Done if `transientFailure` did not check the context first.
  The empty-task-branch and unusable-origin skips have rows with no `RequiresPullRequest` call.

## Rejected

- An unreadable rule with no record still lands by a direct push GitHub may refuse.
  The decision record asks that lym never chooses a direct push to a branch that requires a pull request, but an unreadable rule leaves the requirement unknown, and `doc.go` states this bound.
- The Finalize and PR-Review rows each repeat the token and transient rule-read rows.
  They exercise one shared helper through two different callers, so each row pins its own wiring and neither is redundant.
