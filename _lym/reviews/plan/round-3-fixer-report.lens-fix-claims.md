```yaml
lens: fix-claims
outcome: reported
```

fix-claims lens: I checked every claim the fix added against the code at HEAD.

No findings.

- **Card 06, TestRemove row:** The row says the record file's lock is a blocking flock and that a `Settle` called inside it never returns. Both hold. `state.UpdateJSON` takes `lock.AcquireWriteLock` (internal/state/state.go:154), which is a blocking `gofrs/flock` (internal/lock/lock.go:30-31). The doc comment at internal/state/state.go:147 says a nested acquire on the same path "hangs rather than failing".
- **Card 06, "both present or both gone":** This holds under the card's own design. Steps 2 and 3 of `Remove` run together under the lock, and `Settle` holds the same lock across its read. The goroutine is started inside `RemoveEntry` while the lock is held, so it gets the lock only after `Remove` releases it. It then reads entry and record both gone, which is consistent with the row's claim.
- **Card 11, new TestAdopt row:** The row says a records branch's `Worktree` is the records sibling and its `CodeWorktree` is the pair's code worktree. That matches `originTargets` (internal/fabricengine/originreconcile.go:646: `worktree: sibling, codeWorktree: worktree`) and the `BranchOutcome` doc at :163. A code branch has both fields equal (:642), so the row tells the two rules apart.

Tool calls:
1. One Bash call that read `UpdateJSON`'s lock code, internal/lock's acquire functions, and `BranchOutcome` plus `originTargets`' target construction. My context held only the `CodeWorktree` grep lines and the state doc line, not the acquire path or the records-side `worktree:` assignment.
