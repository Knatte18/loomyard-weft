```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set-premise searches for F1, F4 and F5 and check the fix covers every member.

**Findings**

1. LOW, `internal/fabricengine/add.go:890` (the rollback's warp-side `removeGitWorktree`) and `internal/fabricengine/prune.go:284`.
   - Both call `removeGitWorktree` without `finishGitWorktreeRemoval`.
   - A directory refilled after git's clean exit, or deregistered and left by a git failure, stays on disk on those two paths.
   - The review named neither site, and `prune.go` has its own registration checks at lines 205 and 293 that I did not read.
   - The summary says Add's rollback "gets the same finish", but the finish covers only its weft side (`add.go:852`).
   - If that wording reads as covering the warp side, it is slightly overbroad.

**Checked, no gap**

- F1: `removeWeftWorktree` has three call sites (`add.go:852`, `remove.go:539`, `weftwiring.go:210` as its definition). All pass `leftover`, and `TestAddRollback_FinishesRefilledWeftWorktree` pins the rollback one.
- F4: `finishGitWorktreeRemoval` has four call sites (`remove.go:469`, `remove.go:1020`, `weftwiring.go:210`, plus its definition).
  - `remove.go:1020` passes `gitErr` nil, so the new list-failure branch never runs there.
  - The other `isRegisteredLinkedWorktreeIn` callers (`prune.go:205`, `prune.go:293`, `destroy.go:607`, `destroy.go:720`, `destroy.go:758`, `remove.go:1094`) keep their conservative false, unchanged.
- F5: the new clean-exit wording appears only at `destroy.go:1426`. No test or doc string-matches the old wording; the `remove.go:1050` message belongs to a different path and is untouched.

**Tool calls**
1. `grep` for `removeWeftWorktree(`, `finishGitWorktreeRemoval(`, `isRegisteredLinkedWorktreeIn(` and `removeGitWorktree(` in `internal`: F1 and F4 call-site sets, which my context did not hold.
2. `grep` for the old and new clean-exit message text in `internal`, `docs` and `_lym/landing`: F5 pins and cross-references, which my context did not hold.
