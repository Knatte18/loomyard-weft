```yaml
lens: code-correctness
outcome: reported
```

Task: lens code-correctness, running each added or changed branch with a concrete input.

**Per-branch list**
- `ResolvePullRequest`, `parentRuleRequired` true, `off` parent, empty field, GitHub origin, commits: Open, reason names the rule. Correct.
- `ResolvePullRequest`, `on` parent, field `off`, rule true: Warn, Open. Correct.
- `ResolvePullRequest`, rule true, missing origin fact: Halt with no cancel clause, because `required` suppresses it. Correct.
- `ResolvePullRequest`, rule true, empty task branch: No. Correct.
- `resolveWithParentRule`, first pass Open or Halt, or no commits, or a missing origin fact: no read. Correct.
- `resolveWithParentRule`, rule read true: second resolve returns Open. Correct.
- `resolveWithParentRule`, rule read false: first decision returned. Correct.
- `resolveWithParentRule`, transient error: returned as an error, so the driver re-steps. Correct.
- `resolveWithParentRule`, 404 error: Warn, then no rule. Correct as documented, but see F1.
- `resolveWithParentRule`, token failure under a cancelled ctx: skips `transientFailure`, so the cancel is swallowed here and only surfaces at the next check. Harmless.
- `Client.RequiresPullRequest`, `guarded`, `linear`, `bare` and an unknown branch: correct. `rules` is only nil on a null body, and the nil guard covers that. A branch name with `#` or `%` is not URL-escaped by go-github, which looks unlikely to matter.
- `Client.Reopen`: sends `{"state":"open"}` only. Correct.
- `PRReview.endReview`: skips the write when `ReviewEnded` is already set, otherwise records it and returns Done. Correct.
- `judge`, Closed branch (`ReviewEnded` clean, `ReviewEnded` dirty, `Continue` with a clean gated head): correct.
- `judge`, Open branch, `Continue` with a clean gated head: `endReview` leaves the PR open. Correct.
- `judge`, Open branch, `ReviewEnded` set, tree dirty: **wrong**, see F2.
- `openPullRequest`, `ReviewEnded` and head moved: removes the record, then `FindOpen` finds the still-open PR and records afresh with `Continue` cleared. Correct.
- `pullRequestLanding`, Open: merge on GitHub. Correct.
- `pullRequestLanding`, Closed without `ReviewEnded`: halt. Correct.
- `pullRequestLanding`, Closed with `ReviewEnded`, `Reopen` fails: `githubFailure` re-steps on a transient error and halts otherwise. Nothing is merged. Correct.
- `pullRequestLanding`, Closed with `ReviewEnded`, `Reopen` succeeds, a later Finalize step halts: the PR stays open and the re-entry takes the Open case. Correct.
- `pullRequestLanding`, decision No with a record: drops the PR and lands locally. Correct, but see F1.
- `mergeRefusalReason`: now offers only the approval route, `lym loom resume` or `lym batten cancel`. Correct.

**Findings**

F1. MEDIUM, design. `internal/landingshed/pullrequestmode.go:106-127` with `internal/landingshed/finalize_pr.go:141-153`.
- A non-transient rules-read failure reads as "no rule", and Finalize cannot tell that from "no rule exists".
- PR-Review opened a PR because the rule required one, so a record exists. If Finalize's read then fails non-transiently, say a 403 or 404 from the rules endpoint, the decision is No.
- `dropRecordedPullRequest` then closes the open PR, removes the record, and Finalize squashes locally and pushes directly to a branch that requires a PR. That is the bug in the goal.
- Fix: when a PR record exists, treat an unreadable rule as an error stop or halt instead of a soft no. Alternatively, keep the PR and halt whenever the rule read fails and a record exists.

F2. MEDIUM, design. `internal/landingshed/prreview.go:684-690`, with `prreview.go:216-220` and `prsession.go:210`.
- Ending the review no longer closes the PR, so a PR-Review re-entry (resume or `goto PR-Review`) can find an open PR whose record has `ReviewEnded` and `Continue` both true.
- On that path `openPullRequest` waits without a session, and `resumeSession` returns early for `ReviewEnded`.
- The Open case of `judge` only checks `Continue`. With a dirty tree or an ungated head it returns "keep waiting", and nobody is there to clean the tree. The row polls forever.
- The old closed-PR path returned Stuck via `dirtyEndReason` here.
- Fix: in the Open case, treat `rec.ReviewEnded` like the Closed case. Clean returns Done, dirty returns Stuck with `dirtyEndReason`.

F3. LOW, design. `internal/landingshed/finalize_pr.go:170-177` and `prreview.go:677-678`.
- `ReviewEnded` is the only evidence that the operator ended the review.
- A PR that someone closed on GitHub for another reason, such as abandoning it, while the continue flag stood or after the ending, is reopened and merged.
- This is the accepted behavior in the summary. It leaves no way to tell the cases apart, so I am noting it only.

**Tool calls:** none. My context already held every file I needed.
