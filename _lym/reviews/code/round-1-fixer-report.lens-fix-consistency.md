```yaml
lens: fix-consistency
outcome: reported
```

Task: check the change set through the fix-consistency lens (siblings, cross-references, twins left behind).

Findings:

1. **LOW** — `internal/fabricengine/doc.go:592-596`, the `PairTaskBranch` paragraph.
   - **Summary:** The package doc was not updated for the new `PairTaskBranchOrOrigin`.
   - **Evidence:** The paragraph describes `PairTaskBranch` and `TaskBranchOnOrigin` but not `PairTaskBranchOrOrigin`. That method is now the call that `internal/pairteardown/pullrequest.go` and `internal/battencli/readopt.go` make for a pair that may already be gone. The paragraph also says every step reads branches "from the pair", which no longer describes those two callers.
   - **Check:** `grep PairTaskBranchOrOrigin internal/fabricengine/doc.go` returns nothing.

2. **LOW** — same `doc.go` paragraph, which lists the worktree, records-worktree and local-branch steps.
   - **Summary:** It does not mention that an unresolvable records repository now skips the records-worktree step.
   - **Evidence:** `pairTaskBranchLocal` in `pairbranch.go` now falls through to the local-branch step in that case. The prose gives no hint of this, so it matches the code only by omission.

The other places I checked agree with the change:
- `Remove`'s doc ("with nothing of the pair left locally, origin names them") is consistent with `refuseRemoval` and `localPartRemains`.
- `refuseWhenNothingRemains` has a single caller, `refuseRemoval`, and that caller handles the new return value.
- `pairparts.go` is unchanged. I read `TaskBranchOnOrigin` and `TaskBranchSlugs` as consistent with the new code and did not check them further.
- The remaining `PairTaskBranch` callers are `wire.go:1032`, `remove.go:540` and `taskbranchdiscard.go:49`. `wire.go:1032` only runs while the task worktree is present, so the local read is enough. The other two do their own origin fallback.
- No other occurrence of `ownedManagedBranch(l, "")` remains.

Tool calls made:
1. `grep` for `PairTaskBranch` callers, `ownedManagedBranch(l, "")` and `branch_prefix` in `doc.go`. My context did not show callers outside the files the fixer read, or the doc text.
2. `sed` of `battencli/wire.go:1015-1045` and `doc.go:588-598`. I needed the `wire.go` caller and the `doc.go` text, which were not in context.
3. `grep` for other `PairTaskBranch(` callers outside `fabricengine`. This confirmed `wire.go:1032` is the only one.
