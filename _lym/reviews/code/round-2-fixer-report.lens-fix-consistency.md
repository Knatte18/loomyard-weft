```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1–F4 change set for stale siblings and unchanged twins (fix-consistency lens).

**Findings**

1. **LOW** — `internal/fabricengine/remove.go:684`, the `refuseRemoval` doc comment.
   - **Summary:** The comment still gives the old condition for asking origin.
   - **Evidence:** It reads "returns the name origin holds the pair under when nothing of the pair is left locally, else """. F3 changed `refuseWhenNothingRemains` to ask origin once no worktree and no local branch is left (`branchPartRemains`), so a launcher-only pair is now asked too. The fixer updated the `Remove` doc, the `removePair` comment and `refuseWhenNothingRemains`, but not this twin.

2. **LOW** — `internal/fabricengine/pairbranch_integration_test.go`, the file header and the `TestPairTaskBranch_ReadsThePairItself` doc.
   - **Summary:** Both still describe only the empty-prefix case.
   - **Evidence:** The file header says pairs are reached "by their bare-slug branches". The test doc says "over a pair made under the empty prefix". F2 added rows for `earlierBranchPrefix` (`TaskWorktreeOnAnEarlierPrefixBranchIsRead` and `RecordsWorktreeOnAnEarlierPrefixBranchIsRead`), which rename the pair's branches to `earlier/<slug>`.

3. **LOW** — `internal/fabricengine/pairbranch.go:24`, the `slugOfBranch` doc and its callers.
   - **Summary:** F2 let `PairTaskBranch` recognise a pair under any earlier prefix, but the branch-to-slug twin still recognises only the current prefix and the bare slug.
   - **Evidence:** `slugOfBranch` is `strings.TrimPrefix(branch, cfg.BranchPrefix)`. It is called from `cleanup.go:273`, `cleanupremotewarp.go:147` and `:197`, and `pairparts.go:74`. After a second prefix change, a branch `earlier/<slug>` yields the slug `earlier/<slug>` in those paths.
   - The decision record's chosen fix names these steps ("accept a branch with the current prefix and one without"), so this is arguably within the narrower recorded scope.
   - `pairBranchNames`, `originTaskBranch`, `originPairBranch` and `addparent.go:48` also take only the current prefix and the bare slug. That fits the worktree-as-source rule, because an earlier-prefix pair with nothing local is not found. It means a worktree-less earlier-prefix pair is not found on origin.

No BLOCKING or MEDIUM findings. The `ErrPairNotFound` doc, the `Remove` doc, `RemoveRefusal`, `pairState` and the F1 constant swap agree with the code. The `localPartRemains` removal left no caller behind.

**Tool calls (1)**
- `grep` for `pairBranchNames`, `slugOfBranch`, `localPartRemains` and "nothing of the pair is left locally" under `internal`, non-test files. My context lacked the callers outside the files I had read and any twin comments.
