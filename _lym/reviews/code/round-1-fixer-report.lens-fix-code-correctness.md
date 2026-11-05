```yaml
lens: fix-code-correctness
outcome: reported
```

Task: fix-code-correctness lens over the F1–F6 change set (commits 87221e29..31e251d0).

Branches run:
- `clearStray` with removal `ErrStrayPathKept`, delete returns (true, nil): result carries `StrayPathDeleted=true`, `StrayPath=""`, error nil. Correct.
- `clearStray` with a failed delete: returns early, `StrayPathDeleted` stays false. Correct.
- `removePairAndBranch` copies `StrayPathDeleted` before the error check. It is only set on the success path, so no wrong value reaches the report.
- `cancelEnvelope`: the key and help text were updated together.
- `RemoveStrayTaskPath` with `"nested/slug"`, the prime name, a registered worktree or a `.git` entry: each refusal is reached and nothing is deleted.

Test checks against the defect:
- F2 `TestTeardownRequest_…`: fails if `ClearStray` is dropped from `teardownRequest`.
- F3 plain-directory row: fails if `StrayPath` is unmapped or missing from `Any()`. The registered-worktree case is covered by the existing `got != all` row.
- F4 `MemberHoldsAPairOnlyThroughARegisteredWorktree`: the old `os.Stat`/`IsDir` reading gives `HasPair` true for `t`, so it fails with the defect.
- F5 invalid-slug row: without the slug validation the delete runs and the directory is gone, so it fails.
- F5 prime-slug row: without `refusePrimeSlug` the `.git` guard answers instead, so the `"prime worktree"` text fails.
- F5 remote-failure test: swapping the order of `out.err` and `strayErr` makes the error wrap `ErrStrayPathKept`, so it fails.
- F6 rows (`wantDeleted`, `StrayDirectoryDeleted`): fail if the field is not threaded through.

Findings:

1. LOW — `remove.go` deferred respawn, the revert in commit 31e251d0.
   - The fixer dropped the `!errors.Is(err, ErrStrayPathKept)` skip because the recorder showed no spawn without it. That shows no spawn happens, not that nothing else does.
   - The common `ClearStray` flow is `Remove` returning `ErrStrayPathKept` with the directory present, then `clearStray` deleting it.
   - The deferred `respawnPairPush(l, slug)` now runs on that error. It takes the `pathPresent(taskWorktree)` branch and calls `lymcwd.ResolveWorktree` on a plain directory.
   - That most likely fails and logs the `could not respawn the pair's push after a failed remove` warning on a flow that then succeeds.
   - The skip was also not what F5 asked for. F5 asked for a test of it, not its removal.
   - I could not confirm `resolveCore`'s behaviour on a plain directory from what I read.
   - The fixer-report should record "fixed differently: removed the unobservable skip". The warning side effect is the cost.

2. LOW — F6 reports only the deletion `clearStray` makes.
   - A refilled directory that `Remove` itself deletes through its own `removeWorktreeLeftover` sets neither `StrayPath` nor `StrayPathDeleted`.
   - That case is the pair's own late-writer cleanup, so it is arguably out of F6's scope.
   - The `localMode` and `!out.ran` returns of `strayErr` still have no row. The review's F5 named only the remote-failure ordering, so I raise it only as an observation.

Tool calls made:
1. `grep -n "func ResolveWorktree" -A 40 internal/lymcwd/*.go`, to see how `respawnPairPush` treats a plain directory; my context did not show `lymcwd`.
