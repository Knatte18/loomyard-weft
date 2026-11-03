```yaml
lens: code-correctness
outcome: reported
```

Task: lens code-correctness. I ran each changed function and branch with one concrete input and judged the result.

**Per-function list**

- `finishGitWorktreeRemoval`, git exit clean, directory gone: returns nil. Correct.
- `finishGitWorktreeRemoval`, git exit clean, directory refilled, removal succeeds: nil, with `KindPathRemoved` recorded. Correct.
- `finishGitWorktreeRemoval`, git exit clean, directory refilled, removal fails with `ENOTEMPTY` three times: returns an error wrapping the cause with `%w`, so `errors.Is` works. Correct.
- `finishGitWorktreeRemoval`, `gitErr` is a `*destructiveRefusal` (gate refused before git ran, including a stray unregistered path or a weft path that is absent): `errors.As` on `GitError` is false, so `gitErr` comes back unchanged and `errors.As` on the refusal still works. Correct.
- `finishGitWorktreeRemoval`, `gitErr` is a `GitError` and the target is still registered (dirty without force, locked): returns `gitErr` unchanged. Correct.
- `finishGitWorktreeRemoval`, `gitErr` is a `GitError`, the target is deregistered and absent: returns `gitErr`, as the old code did. The callers then decide failure from `pathPresent`: `removeLocalWeft` tolerates it, and `removeSibling` tolerates it via `os.Stat`. Correct.
- `finishGitWorktreeRemoval`, `gitErr` is a `GitError`, the target is deregistered and present, removal succeeds: returns nil, so `removeWeftWorktree` leaves `firstErr` unset and `removeLocalWeft` does not fail. Correct.
- `finishGitWorktreeRemoval`, same case but removal fails: wraps with `%w` on the leftover error only and `%v` on `gitErr`. The `GitError` can no longer be reached by `errors.As` through this wrapper. No caller of the three sibling sites uses that, and the old warp path had the same shape. Correct.
- `removeWarpWorktreeDir` clean path: now calls the helper. Container, repoDir and the `isRegisteredLinkedWorktree` predicate match the old nonzero path (`isRegisteredLinkedWorktreeIn(l.WorktreePath(), target)`), so the two paths cannot disagree on ownership. Routing the old nonzero path through the helper would only change its message wording, so leaving it alone is correct.
- `removeLocalWeft`: `removeErr != nil && pathPresent` still decides failure. A nil return with the directory gone cannot leave a stale error. `RemoveStepSiblingWorktree` follows `hadWorktree`, so a finished removal is reported. Correct.
- `removeSibling`: step reporting reads `pathPresent` after `removeWeftWorktree`, so a finished removal is listed. Correct.
- `removeWeftWorktree` and `rollbackAdd` (`force=true`, `weftReused=false`): a deregistered weft directory left by git is now deleted. The ownership proof is the gate (`ownedRegisteredLinkedWorktree`) that passed just before git ran, and `weftReused` is checked by the caller, so a reused or adopted weft worktree never reaches this code. Correct.
- Containment: `RecordsWorktreePath` is `weftname.SiblingPath(l.HubPath, slug)` and `WorktreePath` is likewise a hub child, so `removeContainedPath` should not refuse a legitimate sibling. Correct as far as I read; I did not read `weftname.SiblingPath` itself.
- `leftoverSeam.refill`, `SetLeftoverSeamForTest`: a nil `refill` is a no-op in production. The setter now assigns fields, so an earlier `SetLeftoverRefillForTest` survives a later seam call, and tests that only set `remove` and `sleep` are unaffected. Correct.

**Findings**

1. **LOW, design.** `internal/fabricengine/destroy.go:1410`. `finishGitWorktreeRemoval` takes "the target is not registered" from `isRegisteredLinkedWorktreeIn`. That function answers false when `List` fails to enumerate (`remove.go:1106-1110`), and its doc calls false "the conservative direction". Here false is the direction that licenses a recursive delete.
   - Case I ran: git exits nonzero on a dirty, registered sibling, then `List` fails transiently. The helper treats the target as deregistered and deletes the worktree. The task side already had this shape in `removeWarpWorktreeDir`, but this diff extends it to the sibling side, including the `force=false` `removeLocalWeft` and `removeSibling` calls.
   - Likelihood is low, since `List` rarely fails right after a failed `git worktree remove`.
   - Fix direction: a `List` failure should return `gitErr` unchanged.

2. **LOW, design.** `internal/fabricengine/remove.go:324-326` (new clean-exit branch at `remove.go:1020`). A failed finish after a clean git exit now makes `Remove` return `RemoveResult{}` and an error, where before it reported success. That leaves the pair half-torn: the sibling worktree, the warp branch and the launchers are untouched.
   - On retry the task path is deregistered but present, so `inspectPair` should report it as `strayPath`. `strayPath` is documented as reported and never deleted (`remove.go:564`), so a retry cannot finish the directory.
   - The error text says the directory was refilled, but gives no way forward. `PATTERN-refusal-way-forward` covers webster, shed and loom refusals, so I am not claiming it applies here.
   - The old nonzero path has the same limitation, so this is consistent with existing behaviour, not a regression.
   - I did not read `inspectPair`; I took this from `pairState`'s doc comment.

No other defect found in my lens.

**Tool calls:** none. The context already held every function and call site I needed.
