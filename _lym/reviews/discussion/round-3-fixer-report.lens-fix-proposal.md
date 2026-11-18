```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the review proposals the fixer carried out for F30 and F31, as the fix-proposal lens.

Proposals challenged: F30 (commit first from a temporary index, then abort) and F31 (keep the already-in-progress refusal; `mergeresolve` is the only abort-first path).

1. **MEDIUM.** Changed text: D5, "`mergeresolve`'s abort of an in-progress merge before its attempt stays as it is, and is the only path that aborts a merge before merging again."
   The claim came from the reviewer's proposal, and the code does not hold it.
   The landing's pre-merge step also aborts the row's own parked merge-in before merging.
   Evidence: `internal/loomcli/landingdeps.go:202-214` says "The pre-merge step probes the pair's merge state. It aborts the row's own parked merge-in and stops the run's conflict sessions first", and its `AbortMerge` calls `f.MergeAbort()`.
   Fix: name both paths, the resolver's and the landing pre-merge step's abort of its own parked merge-in, or drop "only".

2. **MEDIUM.** Changed text: D5, "`lym fabric merge --continue` on a merge stopped before step 3 refuses … way forward: `lym fabric merge --abort`".
   A sibling way forward still sends the operator to `--continue` for any parked fabric merge.
   Finalize's refusal for a parent pair holding a fabric merge offers "concludes it with \"lym fabric merge --continue\" or discards it with \"lym fabric merge --abort\"".
   A pre-step-3 merge has a merge record, so it can read as `MidMergeParked`, and the operator gets a `--continue` that the new rule refuses.
   Evidence: `internal/landingshed/finalize.go:621-633` and `parkedMergeWayForward` just below.
   The record should say that Finalize's parked-merge way forward, and `fabricengine.MidMerge`'s state, tell a merge stopped before step 3 apart and name only `--abort` for it.

3. **MEDIUM.** Changed text: D4, "with the worktree's content of every records path whose index entry equals HEAD's, and HEAD's version of every other records path".
   Concrete case: a run's first transition after the crash writes a new status file, which is absent from HEAD and from the index.
   Read literally, "index entry equals HEAD's" picks only paths that have entries, so a plan writer enumerating index or HEAD paths leaves the new file out of the commit.
   `git reset --hard` keeps untracked files, so the file survives the abort but is never committed, and the run acts past a transition whose records commit lacks it.
   That is the F30 premise again, for the new-file case.
   Fix: state that a records path absent from both HEAD and the index, an untracked new file, is taken from the worktree.

No other case found against F30's rule.
The source-only path, the unmerged path, a step 3 left half done and a deleted path each come out as the record states.

Further tool calls:
- `internal/landingshed/finalize.go:600-650` and a grep for `MergeAbort()` callers: my context lacked the other abort paths and the parked-merge way-forward text.
- `internal/loomcli/landingdeps.go:185-229`: my context lacked what the landing's `AbortMerge` caller does.
