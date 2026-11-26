```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of the F1 change set (commit f3c8b0187) against its surrounding text and twins.

1. **MEDIUM.** Changed text: the `PRReview` doc line "A re-entry on a review not ended halts before any push when the branch on GitHub holds a commit pushed by hand" (`internal/landingshed/prreview.go:39`), the `openPullRequest` doc "a hand commit on its branch on GitHub halts with nothing pushed" (`:164`), and `docs/overview.md:1404-1405` "halts the row, at a re-entry before any push".
   - **Summary:** all three claim more than the code does.
   - **Evidence:** the entry check runs only under `if status.State == pullrequest.StateOpen`. For a recorded, unended review whose pull request is closed or merged on GitHub and whose head equals `rec.GatedHead`, the new block goes on to `pushTask` with no foreign-head check, whatever the branch holds. The claim needs "whose pull request GitHub reports open".

2. **LOW.** Changed behaviour: `openPullRequest` now calls `foreignHeadHalt`. Its twin text was left as it was.
   - **Summary:** the function's doc and halt reason still describe only the wait.
   - **Evidence:** the doc says "halts the wait when…", "pushed to the branch by hand while the review waits" and "It returns nil while the wait goes on" (`prreview.go:699-704`). The reason says "pushed there by hand while the review waited" (`:716`). At entry no wait has started, and in the review's own scenario (closed-PR halt, reopen, push a fix, resume) the commit was pushed while the run was halted. The exact-reason rows at `prreview_wait_test.go:449` and `:495` pin that wording, and the first of them now runs only the entry path.

Outside my lens, in one sentence: if the new comment "A push can drop a hand commit" holds, the pushes on the closed and review-ended paths, and `gateHead`'s push after a long gate, can still drop one.

Tool calls:
- One `git grep` over `internal`, `docs`, `contracts`, `PATTERN.md` and `README.md` for other hand-commit and push-rejection statements; my context held only the files and greps the fixer had read, not a repo-wide search for these phrasings.
