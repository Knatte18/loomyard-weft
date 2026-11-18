```yaml
lens: fix-claims
outcome: reported
```

fix-claims: one finding.

- **MEDIUM.** Changed text in D5's crash paragraph of `_lym/discussion/decision-record.md`: "`mergeresolve`'s abort of an in-progress merge before its attempt stays as it is, and is the only path that aborts a merge before merging again."
  - Summary: the claim that `mergeresolve` is the only path is false. The landing's pre-merge step also aborts the row's own parked merge-in before it merges.
  - Evidence: `internal/loomcli/landingdeps.go:201-213` has the comment "The pre-merge step probes the pair's merge state. It aborts the row's own parked merge-in and stops the run's conflict sessions first." Its `AbortMerge` closure there calls `f.MergeAbort()`.
  - The claim came from the reviewer's proposed fix for F31.
  - The other callers of `MergeAbort` are `internal/fabriccli/merge_verbs.go:229` (`--abort`) and `internal/mergeresolve/mergeresolve.go:46,197`.

The other claims I checked hold:
- `MergeAbort` resets hard to `st.WarpStart` through `resetMergeSides` (`internal/fabricengine/mergelifecycle.go:370`, `destroy.go:2492-2501`), so moving `WarpStart` first keeps the records commit.
- The verbs' already-in-progress refusal fires whenever the merge record exists, before anything mutates (`internal/fabricengine/mergeguards.go:168-178`, `merge.go:66-73`), so a parked merge's resolutions stay.
- An unmerged records path can only be one the source changed.

Tool calls:
1. Grepped the `MergeAbort` callers and read `mergeInProgressReason`. My context had neither the full caller set nor the pre-lock guard.
2. Read `internal/loomcli/landingdeps.go:190-235` to confirm what the landing's abort caller does. My context did not show that file.
