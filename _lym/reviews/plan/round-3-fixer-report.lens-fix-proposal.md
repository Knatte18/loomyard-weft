```yaml
lens: fix-proposal
outcome: reported
```

I checked the reviewer's proposals the fixer carried out as proposed: F1 (card 6 TestRemove lock-race row) and F2 (card 11 TestAdopt records-branch warning row).

**[MEDIUM] F1: the goroutine row cannot reliably catch the regression it exists for**
- Changed text: `_lym/plan/06-front-task-package.md`, the TestRemove row. The fake `RemoveEntry` "starts `Settle` in a goroutine while the removal holds the lock … records nothing and runs no action whichever way the goroutine interleaves with the removal, since `Settle` reads the entry and record either both present or both gone".
- What is wrong: the row is the only test of the race the lock prevents. Once `Settle` runs in a goroutine, an implementation that drops the record and removes the entry outside the lock fails the row only when the goroutine happens to run between the drop and the removal. Otherwise it passes. So the row catches a missing lock only some of the time.
  - The interleaving sentence also misleads. Under a correct implementation the goroutine starts after the drop, while the lock is held, so `Settle` can only acquire the lock after `Remove` releases it. It always reads both gone, never "both present".
  - The review's own Issue text claims the goroutine "gives the intended coverage deterministically". That holds for a correct implementation only, not for catching the defect.
- Evidence: card 6 line 68 puts steps 2 and 3 under the record file's lock. `internal/lock/lock.go:40` has `TryAcquireWriteLock`, which returns false without blocking while the lock is held, and `internal/lock/lock_test.go:74` `TestTryAcquireWriteLock` covers it.
- A deterministic change: the fake `RemoveEntry` asserts that `lock.TryAcquireWriteLock` on the record's lock path reports the lock as held, which pins the lock directly. The goroutine `Settle` then asserts that nothing is recorded and no action runs once it returns. Drop the "both present" clause, or limit it to a `Settle` that wins the lock before `Remove` takes it, which this row never sets up.

**F2: no finding**
- The new row picks state diverged for a records branch. For records, `judgeBranch` yields diverged only as a replay that stopped on a conflict (`originreconcile.go:136`). It yields ahead, needs_replay or level otherwise, and never rewritten.
- A diverged branch is a stop state (`stops()`, line 155), and `classifyEntries` marks the pair's entry stopped. The existing sibling row "a diverged branch of an adopted beam's pair … is a warning" has the same property, so the new row is consistent with card 11's existing reading of "no stop of that slug already reported".
- The `Worktree` / `CodeWorktree` choice matches `originTargets` line 646.

Tool calls:
1. Grepped `originreconcile.go` for the records states and `internal/lock` for a try-lock API. My context lacked which states a records branch can reach and whether a non-blocking lock probe exists.
2. Read `judgeBranch` and `Summary`. I needed to know whether a records branch can read rewritten or diverged.
3. Read `stops()`, `WayForward` for diverged, and card 11's exclusion lines. I needed to know whether diverged records is a stop.
4. Read `classifyEntries`. I needed to know whether a diverged records branch stops its pair's entry.
