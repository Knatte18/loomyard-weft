```yaml
lens: fix-claims
outcome: reported
```

I checked the fix's claims against the code at HEAD.

no findings

These claims hold:
- **`Pool.TryAcquire`'s one production caller:** it is `testmapaudit.go:52`. The helpers it reaches (`prepare`, `readLimits`, `lockIsHeld`, `drainLockPath`, `tryAcquireAny`) are still called from `Pool.Acquire` (`pool.go:145-151`) and elsewhere in `pool.go`. Deleting it leaves no newly unreachable function.
- **`FuncIndex.ProfiledFuncs`'s one production caller:** it is `testmappackage.go:475`, inside `newBlockTable`. That branch fills `blockTable.profiled`, which only the kept-entry self-check reads (`testmappackage.go:734`), so calling it the self-check branch is accurate. Its helper `funcsHolding` stays reached through `CoveredFuncs`.
- **Computed scope with a nil green record:** `diffBaseGroups` (`scope.go:393-397`) takes the merge base. With no parent branch, `parentMergeBase` returns not-found (`scope.go:675-677`) and the tier lands in `wholeTiers`. Nothing is narrowed, so `computeScope` returns `scope{whole: true}` (`scope.go:270-271`), which is every package `--all` builds. That matches "the whole module when the worktree has no parent branch" and the new subtest's expected outcome.
- **"the map run reads no green record at all":** this holds once `runTestMap` stops calling `openGreen`. `openGreen` was the only read path.

Tool calls:
- Read `scope.go:240-298`, `parentMergeBase` and `testmappackage.go:465-480`. My context lacked how the computed scope resolves whole tiers, the no-parent path and the `ProfiledFuncs` call site.
- Grepped `testmappackage.go` for `profiled`/`checked` and the `TestGateTestMap` fixture setup. I needed to confirm the branch feeds the self-check and to see the fixture's tier0 command and the tree's cleanliness.
- Read `pool.go:165-200` and its method list. I needed to see what `TryAcquire` reaches.
- Grepped the non-test callers of `lockIsHeld`, `tryAcquireAny` and `drainLockPath`. I needed to confirm no helper is left reached by nothing once `TryAcquire` is deleted.
