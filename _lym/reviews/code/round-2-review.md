---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/landingshed/finalize_pr.go:209
    summary: A merge parent reopens a closed recorded pull request in pullRequestLanding, before ownerChecks, so a machine that may not own the run calls GitHub, and the widened condition now reopens a pull request no one asked to reopen
    fix: >-
      In pullRequestLanding's StateClosed case, when decision.MergeAtOnce is set, do not call api.Reopen there;
      return a prLanding with merge, atOnce and a new reopen flag set, plus api and record, with a new `reopen bool` field on prLanding
      (doc comment: marks a landing whose closed recorded pull request mergeOnGitHub reopens just before it merges).
      In mergeOnGitHub, after pr.PushTask and the head read and before the merge loop,
      call landing.api.Reopen(ctx, landing.record.Number) when landing.reopen is set,
      judging a failure with fz.githubFailure(ctx, "reopen the pull request", err).
      Keep the existing reopen in pullRequestLanding for the non-MergeAtOnce ReviewEnded case unchanged.
      Update finalize_pr_test.go's two merge-parent closed-record rows so wantCalls and wantOrder show the reopen after task-push,
      and add a row in which ownerChecks halts a merge parent with a closed recorded pull request and wantCalls holds only Status 7.
  - id: F2
    severity: LOW
    class: consistency
    location: internal/landingshed/doc.go:259
    summary: doc.go still says only PR-Review writes the operator's ending mark, but openAtOnce in Finalize now writes a record with ReviewEnded set
    fix: >-
      In internal/landingshed/doc.go line 259, replace the closing clause "only PR-Review writes that mark"
      with "PR-Review writes that mark on the operator's continue, and Finalize writes it on the record it opens itself for a yes marked MergeAtOnce".
      Do the same in the ReviewEnded field comment in internal/pullrequest/record.go:35, which now says only the row ended the review.
---

### [MEDIUM] Reopen under a merge parent runs before the owner checks

**Location:** internal/landingshed/finalize_pr.go:209
**Issue:** `landUnderLock` runs `pullRequestLanding` (step 1b2b) before `ownerChecks` (step 1b3), `landCheck` and the lock re-check.
Before this diff the closed-record branch reached `api.Reopen` only for a record carrying the operator's ending.
Line 206 now also lets any closed recorded pull request through when `decision.MergeAtOnce` is set, so a merge parent reopens a pull request on GitHub with no review and no operator ending, ahead of every check that this machine may land the run.
The writer's own summary gives the opposite design for the open: Finalize opens the pull request "after the owner checks and lock re-check ... nothing is pushed or opened on GitHub from a machine that may not land the run".
`PATTERN-owner-machine` ties a run's landing to the machine that owns its entry, and a reopen is an outward GitHub action of that landing.
A foreign-owned run that ownerChecks would halt still has its pull request reopened first, and the halt leaves it open.
The rows `AMergeParentReopensAClosedRecordedPullRequestWithNoOperatorEnding` and `AReopenedPullRequestUnderAMergeParentSquashMergesWhateverTheSquashSetting` show the reopen only in `wantCalls`, with no row where an owner check halts first.
The ended-record reopen under `on` and `required` has the same ordering and predates this diff, so the finding covers only the new merge-parent exposure.

### [LOW] doc.go says only PR-Review writes the ending mark

**Location:** internal/landingshed/doc.go:259
**Issue:** The sentence ends "only PR-Review writes that mark", where the mark is the record's operator's ending.
`openAtOnce` at internal/landingshed/finalize_pr.go:331 now writes `ReviewEnded: true` from Finalize, and doc.go line 263 says so, so lines 259 and 263 disagree.
The `ReviewEnded` field comment in internal/pullrequest/record.go:35 ("the row ended the review on the continue flag") has the same gap but sits outside the diff.

## Rejected

- Focus 1, the `Squash` option: the first parent-side merge and its retry merge both use the one `mergeOpts` value, `Squash: fz.deps.Config.Squash || landing.atOnce`, and `mergeOnGitHub` computes `squash` with the same expression, so the methods cannot differ, including under `squash: false`. The rows `AMergeParentWithNoRecordOpensThePullRequestAndSquashMergesItWhateverTheSquashSetting`, `AReopenedPullRequestUnderAMergeParentSquashMergesWhateverTheSquashSetting` and `AMergedPullRequestUnderAMergeParentLandsThroughASquashingParentSideMerge` set `squashOff` and assert `wantSquashedMerges`. No row reaches the merge-in-required retry under a merge parent, but the retry shares `mergeOpts`, so a row there would add no coverage.
- Focus 2, the `atOnce: decision.MergeAtOnce` flag on the closed-record return: the reopened row with `squashOff` and a record carrying the ending expects `wantGitHubMerge` with squash `true` and `wantSquashedMerges`, so dropping the flag fails it.
- Focus 3, `openAtOnce`: both opening rows set `wantRecord`, and the comparison key covers Number, URL, Base, Head, GatedHead and ReviewEnded.
- Focus 4, halt and refusal texts: under `MergeAtOnce` the closed-record halt and the no-record halt at finalize_pr.go:188 and :207 are unreachable, so no text there names `goto PR-Review` for a merge parent. `protectedParentWayForward` drops the advice to set `pull_request=merge` when `parentMerges` is true, and `mergeRefusalReason` names the repository's settings for a method refusal under a merge parent. `git reset --keep` is explicitly not a denied form in `PATTERN-refusal-way-forward`.
- Declared PATTERN changes: the directive declares none, and the diff touches neither `PATTERN.md` nor `pattern/`. The new Finalize calls to open, reopen and merge a pull request fall under the allowed set in `PATTERN-github-auth`.
