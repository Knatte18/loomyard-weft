```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1 and F2 change set for agreement with its surroundings (lens fix-consistency).

No findings.

- **`doc.go`:** the new lines 259–261 agree with `finalize_pr.go`. The deferred reopen after the task push matches the closed branch and `mergeOnGitHub`. The `ReviewEnded` wording in `doc.go` matches `openAtOnce` and the `record.go:35` comment.
- **Reopen mentions elsewhere:** none left stale. The `finalize.go:172` header says only that a closed pull request is reopened under a parent that takes its pull request at once, which stays true. The `openAtOnce` doc ("reopens it when it was closed") also stays true.
- **Test rows:** the `wantOrder` values and the new `reopen` hook entries are consistent. The on-parent rows reopen before `task-push`, the merge-parent rows after it, and the failed-reopen rows stop at the reopen.
- **`ReviewEnded` readers:** none relies on "only PR-Review writes it". `prsession.go:221`, `prreview.go:229` and `prcontinue.go:32` only read the flag.

Tool calls (1 of 6):
1. Grep for `only PR-Review`, reopen wording and `ReviewEnded` across `internal` and `docs`. My context held only the files the fixer read, not their sibling comments elsewhere in the repo.
