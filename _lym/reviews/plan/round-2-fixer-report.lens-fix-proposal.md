```yaml
lens: fix-proposal
outcome: reported
```

fix-proposal lens: I challenged the reviewer's proposals that the fixer carried out, F1's deletes and F2's nil green record.

**LOW** — card 1, the **Delete:** entry `internal/testcover#FuncIndex.ProfiledFuncs` and its Intent sentence.
After the delete, `funcsHolding(blocks, coveredOnly bool)` has one caller left, `CoveredFuncs`, which always passes `true`.
So the `coveredOnly` parameter and its `if coveredOnly && b.Count == 0` branch become a dead argument that no card removes.
The guard still passes because `funcsHolding` stays reachable through `CoveredFuncs` (`internal/gatecli/testmappackage.go:444`), so nothing fails.
No `unparam` lint config turned up, so no gate fails on it either; it is a stale leftover a reader would notice.
Evidence: `internal/testcover/blockkey.go:162-174`, where `CoveredFuncs` calls `funcsHolding(blocks, true)` and `ProfiledFuncs` calls `funcsHolding(blocks, false)`.

What held up when I challenged it:
- **F1, deleting `Pool.TryAcquire`:** none of its helpers becomes test-only. `prepare`, `readLimits` and `tryAcquireAny` are still called from `Acquire` (`pool.go:131,141,151`), and `lockIsHeld` and `drainLockPath` from other paths (`pool.go:271,295`).
- **F2, passing a nil green record:**
  - The review's premise holds: with HEAD as the base, `DeriveTargets` returns `targetsFor` over an empty change list, not `wholeTargets` (`targets.go:63-94`).
  - So the new `AComputedScopeTakesNoBaseFromTheGreenRecord` row can tell the bug from the fix in the prime, where there is no parent branch: the bug maps nothing and the fix maps the whole module.
  - Nil is already the dirty-worktree path into `computeScope`, `dotnetScope` and `noneScope`, so no caller needs a new guard.

Tool calls:
1. `pool.go:160-200,400-440`, the `funcsHolding`/`CoveredFuncs` callers and a search for `unparam` lint config. My context lacked the bodies of `TryAcquire` and the pool helpers, and the remaining callers of `funcsHolding`.
2. Callers of `tryAcquireAny`, `readLimits` and `prepare`, plus `targets.go:60-100`. My context lacked whether those helpers keep other callers, and how `DeriveTargets` treats an empty diff.
