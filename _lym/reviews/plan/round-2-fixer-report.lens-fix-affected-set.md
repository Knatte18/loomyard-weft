```yaml
lens: fix-affected-set
outcome: reported
```

Lens: fix-affected-set. I re-ran each finding's set search to check the fix covers every member, not only the ones the review named.

List of findings left unchecked: none. F1 is checked for symbols in other packages only; helpers inside `internal/gatecli` that only the deleted methods reach were not swept (cap).

No findings.

- **F1:** I listed every symbol from other packages that the code card 1 deletes reaches (`testmapaudit.go`, `testmapwait.go`, `headSelector`/`headTree`/`planLockedPackage`/`planPackage`/`lockPackage`/`dirtyGoFile`). The ones in `testcover` are `NewSelector`, `PlanUpkeep`, `Selector*`, `Upkeep*`, `MissReason` and `Target`, and card 2 deletes all of them. `gateslot.Pool.TryAcquire` is now on card 1's Delete list. The rest keep production callers after card 1:
  - `lock.TrackedFileLockPath`: `testcover/coveragemap.go:205`, `redundancyreport.go:105`
  - `testcover.ContentHash`: `testfuncs.go:82`, `redundancytests.go:227`
  - `testcover.ScanTestFuncs`: `redundancy.go:585`, `testmapplan.go:180`
  - `ReadMap`, `MapPath` and `ScanSpawns`: the map I/O, which stays
  - `testMapRun.takeSlot`: `testmappackage.go:574`
  - `testmaptree.go` functions: called from `testmappackage.go` (`readPackageTree`, `otherFiles`, `dirtyOwnedFile`, and `changedFiles` in `endJob`, which stays)

  `FuncIndex.funcsHolding` keeps `CoveredFuncs` as its caller, so removing `ProfiledFuncs` leaves no new unreachable function. Its `coveredOnly` parameter becomes always true, but the test-only-code guard doesn't flag that.
- **F2:** The set is the plan text that ties the map run's scope to the green record. In context, card 1's Uses entry and the Intent line 114 sentence have changed. Line 98 ("never reaches … the green subtraction") and the Bound's "green carry" agree with the change, and so do card 3 line 84 ("calls no green subtraction, as card 1 left it") and overview line 28. No site still has the map run reading green.

Tool calls:
1. Grepped the external symbols referenced by the files and methods card 1 deletes; context only had the `TryAcquire`/`ProfiledFuncs` greps.
2. Grepped the production callers of `TrackedFileLockPath`, `ContentHash`, `ScanTestFuncs` and `takeSlot`; context lacked callers for these.
3. Listed `testmaptree.go` functions and their callers, to check that the `headTree` deletion strands none; this wasn't in context.
4. Read which `testmappackage.go` function calls `changedFiles`, to confirm it survives card 1; this wasn't in context.
