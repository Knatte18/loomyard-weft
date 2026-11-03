```yaml
lens: code-tests
outcome: reported
```

**Lens code-tests: per behaviour, the test row that pins it, or none.**

1. **The continue flag on an open PR at a clean, gated head records `ReviewEnded`, leaves the PR open and makes no `Close` call.**
   - Pinned by `prreview_wait_test.go` "ContinueOnAnOpenPullRequestAtTheGatedHeadRecordsTheEndingLeavesItOpenAndIsDone", plus the "TheContinueFlagWrittenBetweenWakes…" and "AChangeToTheRecord…" rows. They assert the exact `wantCalls`, which exclude `Close`, and the exact `wantRecord`.
2. **The continue flag on a closed PR records the ending.**
   - Pinned by "ContinueOnAClosedPullRequestAtTheGatedHeadIsDoneAndRecordsIt" and the ungated variant.
3. **`endReview` skips the record write when `ReviewEnded` is already set. Re-entry on an open PR with `ReviewEnded` ends Done in `judge`'s open case.**
   - None. The only `ReviewEnded` re-entry rows ("ReentryWithAKeptReviewEndedRecord…", "ReviewEndedAtTheGatedHead…", "ReviewEndedOnAClosedPullRequest…") all use a closed PR. An open PR with `ReviewEnded` is a state that did not exist before. See F1.
4. **`openPullRequest` with `ReviewEnded` and a moved head drops the record and starts a new review.**
   - Pinned by "ReviewEndedPastTheGatedHeadOpensANewPullRequest". It uses a closed PR, which is fine for the head-moved branch.
5. **`resumeSession` skips the session for an ended record.**
   - Pinned by "ReentryWithAKeptReviewEndedRecordStartsNone".
6. **Finalize reopens a closed PR whose record carries `ReviewEnded`, then merges on GitHub.**
   - Pinned by "TheOperatorsEndingOnAClosedPullRequestReopensItAndMergesOnGitHub". It checks the call order `Status, Reopen, Merge` and the `task-push,fast-forward,merge,mark,push` order.
7. **A failed reopen halts with GitHub's message and `lym loom resume`, and merges nothing.**
   - Pinned by "AFailedReopenHaltsNamingGitHubsMessageAndMergesNothing".
8. **A closed PR without the ending still halts, including the case with the continue flag only.**
   - Pinned by "YesWithAClosedRecordWithoutTheOperatorsEndingHalts…" and "AContinueFlagWithoutTheRecordedEndingHaltsAsWell".
9. **No Finalize path squashes locally and pushes to a PR-requiring parent.**
   - The old "ClosedForContinueMergesLocally" row is replaced by the reopen row.
10. **The merge-refusal text drops the "lands locally" and `goto PR-Review` wording and names `lym batten cancel`.**
    - Pinned by the pending and other refusal rows, through `wantReason` and `wantNotInReason`.
11. **The `Reopen` client method sends a PATCH with `state: open` and returns the error GitHub answered.**
    - Pinned by `client_test.go` "ReopenSetsTheStateOpen".
12. **`RequiresPullRequest` returns true on a `pull_request` rule on page 2, false for other rules or none, and an error for an unknown branch.**
    - Pinned by `client_test.go` "RequiresPullRequest", with paged, linear, bare and unknown fixtures.
13. **`ResolvePullRequest` with `parentRuleRequired`: opens a PR, warns when the field is off, halts on a missing origin fact without a cancel clause, and stays no on an empty task branch.**
    - Pinned by `pullrequestmode_test.go` "ParentRule".
14. **`resolveWithParentRule` reads the rules for an off parent.**
    - Pinned by PR-Review "OffWithNoFieldReadsTheParentRulesAndIsDone", "OffWithAParentRule…OpensOne" and the Finalize rule row. It also makes no call under an on parent, since the existing rows assert exact `wantCalls`.
15. **A non-transient read failure reads as no rule and logs a Warn.**
    - Pinned by PR-Review "AnUnreadableParentRuleReadsAsNoRuleAndIsDone", which uses a 404 error.
    - The Warn log is not asserted.
16. **A transient read failure is an error stop, in both rows.**
    - Pinned by PR-Review "ATransientParentRuleReadIsAnError" and the Finalize equivalent.
17. **The skip conditions: no rule read when the task branch has no commits, or when an origin fact is missing.**
    - Partly pinned. The local-origin row "FieldOnWithALocalOriginHalts…" has the field on, so the decision is a halt before the read, which proves nothing about the skip.
    - There is no `parentMode: "off"` row with `taskHasCommits` false or a local origin that asserts an empty `wantCalls`. See F2.
18. **A token failure (`tokenFailure`) on the rule read reads as no rule, not as a transient error. An API-open failure goes the same way.**
    - None. The only error values in the rule rows are a 404 and `context.DeadlineExceeded`. See F3.
19. **A cancelled context during the rule read returns a cancel error.**
    - None. See F3.
20. **`pairteardown` fake gained `Reopen` and `RequiresPullRequest` panics.**
    - They only satisfy the interface and need no behaviour test.
21. **The `pr-continue` note and help text, and the `loomcli` comment.**
    - Static prose, exempt.
22. **Test economy: duplicates and per-helper tests.**
    - No duplicate found. The renamed rows replace the old ones.

**Findings**

- **F1 — LOW, design.** `internal/landingshed/prreview_wait_test.go`, near the "ContinueOnAnOpenPullRequest…" rows. Nothing covers re-entry on an open PR whose record already carries `ReviewEnded`. That state is new, reached after Finalize's reopen or after PR-Review ends the review and is re-entered. Its behaviour is `judge`'s open case calling `endReview` and ending Done without a second write. Add a wait row with `seed("h1", withContinue, withReviewEnded)` and an open status, asserting Done with no `Edit` call and an unchanged record. Evidence: `prreview.go:684-690` and `prreview.go:698-706`.
- **F2 — LOW, design.** `internal/landingshed/prreview_test.go` around line 156. The skip conditions of `resolveWithParentRule` have no row asserting a skipped read. Add two `parentMode: "off"` rows in `TestPRReview_Call`, one with an empty task branch and one with a local origin, each with an empty `wantCalls`. Without them, dropping `!taskHasCommits` or the `checkOrigin` early return breaks no test. Evidence: `pullrequestmode.go:372-377`.
- **F3 — MEDIUM, design.** `internal/landingshed/prreview_test.go` around line 168, and `finalize_pr_test.go` around line 105. The documented soft-fail path for an unusable token has no row. `tokenFailure` is checked before `transientFailure` because the HTTP client wraps token errors as transient, so a token failure must read as no rule. A row with `githubclient.ErrTokenUnresolvable` as the `RequiresPullRequest` error should expect Done and no error. Without it, removing `!tokenFailure(err)` would turn every token-less hub into an error stop and no test would fail. A cancelled-context row, expecting a cancel error, is the same gap. Evidence: `pullrequestmode.go:381-388`.

**Tool calls**
1. A grep of the landingshed tests for rows with `parentMode: "off"` and for the token errors, because the diff hunks did not show whether rows for the skip conditions or a token failure existed.
