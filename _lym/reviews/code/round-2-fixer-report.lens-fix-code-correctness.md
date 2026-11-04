```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

I checked these cases by running them step by step. Each holds:

- **Closed record, merge parent, no operator ending:** `pullRequestLanding` returns `reopen` and `atOnce` set. `ownerChecks` runs next. `mergeOnGitHub` then pushes the task, reads the head, reopens and merges, so the API order is Status, Reopen, Merge.
- **Reopen fails under a merge parent:** `githubFailure` returns the same four-value shape `mergeOnGitHub` returns, so it halts with `LandingStopPrefix`. The record stays closed and the next resume takes the same path.
- **Closed record with the operator's ending, not a merge parent:** `Reopen` still runs inside `pullRequestLanding`. The landing it returns has `atOnce` false, the same as before, because `decision.MergeAtOnce` is false there.
- **Caller:** `finalize.go:327` calls `mergeOnGitHub` once and passes `landing` by value. The new `reopen` field is read only inside `mergeOnGitHub`, so the merge-in-required retry cannot reopen again.
- **Owner-halt row:** `newTestDeps` sets no `Reconcile`, `ParentOwner` or `ParentHoldsTaskWork` seam, so the row's `ParentOwner` override is the only thing that halts. The halt reason contains "owned by machine" and opens with `OwnerHaltPrefix`, which the new `wantOwnerHalt` prefix switch expects.
- **Owner-halt row against the old code:** the old code called `Reopen` first, so `wantCalls` of only `Status 7` would fail. The row catches the defect.
- **`reopen` order marker:** `order` is declared before `api`, and the `onCall` hook runs on the Finalize goroutine, so there is no race. The three rows with reopen after the task push, and the new failed-reopen row, show `task-push,reopen`. Under `on`, the rows show `reopen` before `task-push`.

Tool calls I made:
1. A grep of `finalize.go` for `mergeOnGitHub` and `landing.` uses, because my context lacked the `mergeOnGitHub` call site and the `landing.atOnce` uses after it.
2. Two greps in the same command for `newTestDeps` and the `Reconcile`, `ParentOwner` and `ParentHoldsTaskWork` seams, because my context lacked its body. They matched nothing, which is how I know the seams are unset.
