```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the fix added or rewrote in the decision record holds against the code at HEAD.

no findings

Claims checked:
- **D3, build and test bounds.** The coverage build's toolchain carries `BuildTimeout: r.buildTimeout` (`internal/gatecli/testmappackage.go:222`), set from `cfg.RedundancyBuildTimeoutMin` (`internal/gatecli/testmap.go:357`). Tests run with `TestTimeout: r.testTimeout` (`:675`), and `result.TimedOut` maps to `OutcomeTimedOut` (`:686-687`). A build error lands in `build.err` (`:228`, `:234`), which takes the failed-build path.
- **D4, a host whose cap does not cover a process tree.** On such a host `clampParallelToMemory` returns 1 (`internal/gatecli/redundantmutants.go:542-543`). On a host that cannot cap a scope, `internal/gateslot/template.yaml` says "each process gets memory_mb alone as its data segment limit", so "as the tier's own `memory_mb` is there" holds.
- **D6 Bound, gitignored files.** `readPackageTree` reads `CodeTreeAt(rev, …)`, the committed tree (`internal/gatecli/testmaptree.go:28`). `dirtyOwnedFile` reads `Fabric.Status`, which lists only `WorktreeChangedFiles` (`internal/fabricengine/diff.go:130-149`). Neither lists ignored files.
- **D7 and the constraint, every slot ranks above any count.** `Tier.Slots` is documented as "0 means every slot" (`internal/gateslot/tiers.go:42-43`), so the stated ordering is needed and correct.
- **D2 and D8 Rationale, a failed build keeps the entries it read.** `writeFailed` calls `testcover.ReadMap` at write time and keeps its entries when it holds tests (`internal/gatecli/testmappackage.go:863-869`). Those entries can come from another run.

Further tool calls:
1. A grep of `buildTimeout` and `DeadlineExceeded` in the testmap files, `template.yaml` lines 10-20, and the `Status()` definitions. My context showed only line 647, the `lym` build, not the coverage build's bound, the template text or the Status implementation.
2. `internal/fabricengine/diff.go:130-150` and `internal/gatecli/testmappackage.go:215-250`. I needed what `Status` reads and how a coverage build's failure is recorded, and my context had neither.
