```yaml
lens: existing-paths
outcome: reported
```

Existing-paths lens: trace each decision's state through the code that already writes or reads it.

**Paths traced, per decision**
- D6 dobby stop: `runproc.EndRole(dir, HubOwner, RoleDobby, pid)` against a `test-map` record under `HubOwner` — **breaks** (F1).
- D6 restart: a fresh dobby holds no prime-HEAD record and starts a refresh, and a live `test-map` record blocks a second one. Records are pruned on read. — survives.
- D7 config route: a hub base `gates.webster` entry without `reports` goes through `FillMissing` at load. `gates` is a per-name map, not an open map, so the fill descends into each entry. — survives. `reports: []` is present, so it is not filled. — survives.
- D7 config route: `lym config gate --set gates.webster={...}` replaces the entry whole, and load fills `reports` again. — survives.
- D7 step position: the step runs at the lints' site in `gatecli` run, after the tiers. The tier pass is already in the green record, so carry survives. A caught signal while the step runs changes the exit. — **breaks the record's no-result-change claim** (F2).
- D7 step 1: `parentMergeBase` → `Fabric.MergeBase` on the warp checkout, then `ChangedPaths` against the working tree. The merge base and the paths are consistent, and `CodeFileAt` reading the warp checkout fits. — survives.
- D7 step 1: `ChangedPaths` lists deleted paths and both paths of a rename. Step 1 reads them as "changed `_test.go` files". — **misread** (F3).
- D7/D8 report vs `verifytree.Verify` skip: a skipped verify leaves the earlier report, which still matches the same tree. — survives.
- D5 vs Finalize `Fabric.Merge`: the sync and `MergeStart` run in the prime warp checkout. The guard's `pairDirtyReason` would refuse on any file the verb leaves in the worktree, and D5's bound says it writes only map records. `git status` dirt checks run with `GIT_OPTIONAL_LOCKS=0`, so they never take the index lock the merge needs. — survives.
- D5 package-file recheck vs a landing mid-package: handled by the discard rule. — survives.
- D4 map under the hub's `.lym`: I found no existing path that removes or rewrites it. — survives.

**Findings**

F1 — BLOCKING — design — `decision-record.md` D6, lines 117–119, and its acceptance criterion "D6: a dobby stop ends a running refresh" (line 235)
D6 registers the refresh child under `runproc.HubOwner` with its own role, `test-map`. The prime's dobby is itself registered under `HubOwner` as `dobby` (`internal/beamcli/doc.go:353`, `internal/beamcli/dobbydeps.go:77`), and it is stopped with `runproc.EndRole(..., owner, runproc.RoleDobby, ...)` (`internal/orchcli/cli.go:276`). `ender.end` marks every record of that owner whose role it does not end as spared, and skips spared pids and every process below them (`internal/runproc/end.go:160-167`, `:183`). The `EndRole` doc says the same: "Every record of another role, and every process below such a record, is left alone" (`internal/runproc/end.go:72-74`). So a dobby stop spares the `test-map` child. D6's premise that `EndRole` ends every descendant is false, and the acceptance criterion cannot be met as designed. Fix: either the dobby stop also ends `RoleTestMap`, or the child gets no record of its own and stays a plain descendant; the in-flight rule then needs another source.

F2 — LOW — design — `decision-record.md` D7, line 159 ("The step never changes the gate's result")
The step runs at the lints' site. `gatecli` run returns `exitAfterSignal(caught, code)`, which answers `128+n` whenever a signal was caught, whatever the tiers decided (`internal/gatecli/run.go:200-221`, `:709-713`). The step adds up to `redundancy_budget_min` plus the last test's timeout, plus coverage builds and a `lym` build. A cancel or signal that arrives in that window turns a call whose tiers were green into a non-zero exit. The green record is already written, so a re-run carries. D7 should state this exposure, or say that a signal during the step leaves the tiers' code.

F3 — LOW — design — `decision-record.md` D7 step 1, lines 140–144
`impactset.ChangedPaths` returns a deleted file, and a renamed file by both its old and its new path (`internal/impactset/targets.go:99-101`). Step 1 treats every changed `_test.go` path as a file to read now and at the merge base. A deleted file has no current text, and D7 does not say what it does with one. A renamed or moved test file reads as "absent at the merge base", so all of its unchanged tests are judged as new. They fill `redundancy_max_tests`, which is taken in file then name order, and can push the diff's new tests out of judgment. D7 could match by package and test name rather than by file path, and drop deleted paths.

Tool calls:
1. `internal/gatecli/run.go` lint site (lines 180–232), gate-dir cleanup grep and `HubOwner` end grep: needed for where the step sits relative to the exit code and green record; the manifest held only the line numbers of the lint site.
2. `exitAfterSignal` and the busy-quit grep in `internal/gatecli`: needed for how a caught signal sets the exit after the lints' site; not in the manifest.
