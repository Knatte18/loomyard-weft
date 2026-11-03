# Fixer-report, round 1

All four findings fixed as proposed.
Commit c0e00afa holds F1 to F4 together, not one commit each: F1 and F2 both change `prreview.go`, and F1, F3 and F4 all add rows to the same two test files, so the round's edits were made in one pass.
A second commit holds the self-check fixes.

## Changes

- **F1** (MEDIUM): `resolveWithParentRule` takes `recordFound`.
  When the rules read fails for a reason other than a transient or token one and a record exists, it returns a `Halt` built by `githubHaltReason("read GitHub's rules for the parent branch", err)`.
  With no record it keeps the Warn and reads as no rule.
  `Finalize` (`finalize_pr.go`) now reads the record before resolving and passes `found`; its halt carries `LandingStopPrefix`.
  `PRReview.decide` (`prreview.go`) reads the record and passes it as well, so a re-entry cannot close the pull request either.
  Rows: `AnUnreadableParentRuleWithARecordedPullRequestHaltsAndKeepsThePullRequest` in `TestPRReview_Call` and `AnUnreadableParentRuleWithARecordedPullRequestHaltsClosesNothingAndKeepsTheRecord` in `TestFinalize_PullRequest`, which gained a `wantRecordKept` field.
- **F2** (MEDIUM): `judge`'s `StateOpen` case returns `done()` for a record carrying `ReviewEnded` on a clean tree and `stuck(dirtyEndReason)` on a dirty one.
  Rows in `prreview_wait_test.go`: `ReviewEndedOnAnOpenPullRequestOnACleanTreeIsDoneWithoutEditingIt` and `ReviewEndedOnAnOpenPullRequestOnADirtyTreeHaltsNamingTheFiles`.
- **F3** (MEDIUM): token soft-fail rows in `TestPRReview_Call` and `TestFinalize_PullRequest`, and a `ContextCancelledBeforeTheParentRuleReadIsAnErrorWrappingTheCancel` subtest.
- **F4** (LOW): rows `AnEmptyTaskBranchUnderOffDoesNotReadTheParentRules` and `ALocalOriginUnderOffDoesNotReadTheParentRules`, each with no GitHub call.
- Docs kept in step: `doc.go`, the `Finalize` doc in `finalize.go`, the `PRReview` type doc and `resolveWithParentRule`'s doc now say that an unreadable rule with a recorded pull request halts.

## Self-check

Gate after the fork-driven fixes: `lym gate test ./internal/landingshed` passed (0 failed).
All four forks returned a report.

- fix-code-correctness, MEDIUM: the token rows scripted a bare `ErrTokenUnresolvable`, which the transient classifier does not mark transient, so they passed without the `tokenFailure` guard. Fixed: both rows wrap it in `*url.Error` as the existing token rows do.
- fix-code-correctness, LOW: the open `ReviewEnded` verdict does not check `gated`. Disputed (every fix worse), see below.
- fix-consistency, MEDIUM: `doc.go` still said any other failure reads as no rule. Fixed.
- fix-consistency, LOW: `Finalize` doc in `finalize.go` omitted the halt. Fixed.
- fix-consistency, LOW: `PRReview` type doc omitted the halt. Fixed.
- fix-proposal, MEDIUM (F1 tradeoff): see Deferred.
- fix-proposal, LOW (F2, no `gated` check): same finding as the second fix-code-correctness one, merged. Disputed, see below.
- fix-proposal, LOW (cancel row could pass without the cancel check): fixed. The row scripts a non-transient error, so without `cancelErr` the read would Warn and the call end Done.
- fix-affected-set: no findings.

## Disputed

- **Open `ReviewEnded` verdict without a `gated` check** (fix-code-correctness LOW and fix-proposal LOW), ground `every fix worse`.
  The closed case it mirrors (`prreview.go`, `case rec.ReviewEnded && clean: return done()`) has no `gated` check either.
  A `gated` test on the open case would make an ended review with an ungated head wait forever on a clean tree, which is the defect F2 fixes, and any Stuck alternative would give the two states different verdicts.
  Reaching it needs a commit made during an ended review with no session running.

## Deferred

- **F1 tradeoff** (fix-proposal MEDIUM): a record does not prove the rule applied.
  A run whose pull request was opened for the entry's `pull_request: on`, or under a beam.yaml `on` later set to `off`, now also halts when the rules read fails with a 403 or 404.
  The review chose this fail-safe, and the record holds no fact telling the two apart.
  The operator's way out is to fix the token's repository access and run `lym loom resume`, or `lym batten cancel`.
  Whether to record why the pull request was opened is the operator's call.
