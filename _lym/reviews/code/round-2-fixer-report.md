# Fixer-report, code round 2

## Changes

- **F1, fixed as proposed** (commit `fix F1`).
  `pullRequestLanding` no longer calls `Reopen` for a closed recorded pull request under `MergeAtOnce`: it returns a `prLanding` with `merge`, `atOnce` and the new `reopen` field set.
  `mergeOnGitHub` reopens after the task push and head read, before the open and the merge loop, judging a failure with `githubFailure(ctx, "reopen the pull request", err)`.
  The non-`MergeAtOnce` ended-record reopen stays in `pullRequestLanding`.
  Tests in `finalize_pr_test.go`:
  - The two merge-parent closed-record rows now show the reopen after `task-push` in `wantOrder`, through a new `onCall` hook on the fake API, and the `on` rows show it before `task-push`.
  - A new row has the owner check halt a merge parent with a closed recorded pull request, with `wantCalls` of `Status 7` only.
  - A new row covers a failed reopen under a merge parent.
- **F2, fixed as proposed** (commit `fix F2`).
  `doc.go` and the `ReviewEnded` comment in `internal/pullrequest/record.go` say Finalize also writes the ending mark, on the record it opens itself for a `MergeAtOnce` yes.
  `doc.go` also says where the merge-parent reopen runs.
- **Self-check follow-up** (commit `fix F2`, second): the same stale wording in `internal/pullrequest/doc.go` and the `finalize.go` step comment now agrees.

Gate: `lym gate test ./internal/landingshed ./internal/pullrequest` passed before the self-check, tier0 775 passed, 0 failed.
The two self-check wording edits changed comments only and were not re-gated.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: LOW, `internal/pullrequest/doc.go:4` still said only the row ended the review; fixed as proposed.
- fix-affected-set: LOW, `finalize.go:172` comment did not show the new reopen timing; fixed as proposed.

## Disputed

None.

## Deferred

None.
