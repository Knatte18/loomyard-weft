```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced each state the record introduces or changes through the code that already writes or reads it.

**Paths traced, per decision**

- D1, scope value read by the strict `gateslot` loader (`internal/gateslot/config.go:485-492`): survives. The record states that an older binary refuses `coverage`, and AC D1 covers the error text.
- D1, `withGateScope` (`internal/gatecli/scope.go:107-119`): survives once the value is added there. Only `diff` and `all` clear `--card` ids today. That is the implementer's call site, not a finding.
- D1, retired-list migration table (`internal/gateslot/config.go:547-549`): survives. It still writes `diff`, so migrated hubs keep `diff`, which matches the record leaving this hub's `gate.yaml` out.
- D1, the failed-test rerun through `gateslot.FailedTestsCall` (`internal/gateslot/tiers.go:416-447`): survives. It passes explicit targets, which D1 never narrows.
- D1, Darn's `tree_scope` (`internal/loomshed/verifygate.go:97`): survives. It passes explicit targets.
- D2, `testmap` `stale` (`internal/gatecli/testmap.go:450-456`): survives. The record adds the marker and the covered-package rules there.
- D2, `testmap` dirty-file skip `dirtyGoFile` and the post-run check `changedFiles` (`testmap.go:409-417`, `testmappackage.go:213-238`): **breaks**. Both cover only the package's own directory.
- D2, `KeyBlocks` reading block text from the working tree while `fileHashes` hashes HEAD (`blockkey.go:81-99`, `testmap.go:421-439`): **breaks** for covered packages, the same defect as the previous line.
- D2, `testcover.WriteMap` keeping unknown keys of the record it replaces (`coveragemap.go:66-72`): **breaks** the format marker when an older binary rewrites a map.
- D2, the redundancy report's kept set built from `p.m.Tests` and `p.m.Blocks` (`redundancy.go:589-597`): survives. Cross-package keys in kept tests cannot change whether a fresh test's own-package blocks are covered, and D2 already filters to own-package blocks.
- D2, `mapDrift` and `checkMapBlocks` (`redundancy.go:316`, `redundancy.go:333-344`): survive. Own-package `Files` keeps its shape, and the indexes stay inside one table.
- D3, guard step `scanGuards` (`scope.go:535-566`): survives. Importing packages count as inside the scope, so their guard tests never reach the guard step, and D3 adds them explicitly.
- D3, watch skip `targets.Unwatched` (`impactset/targets.go:143-151`): survives. D1 keeps watch skips as under `diff`.
- D6, green `subtract` and `covered`/`argsMatch` (`green.go:333-375`, `green.go:182-229`): survives. An empty-args package entry matches an anchored `-run ^(…)$` request through `isGuardRunFilter`, which is correct because that pass ran everything. A `-run` entry matches only exactly, and `newestWholeAncestor` skips entries with args.
- D6, `recordPass` (`green.go:436-470`): survives with plan work. `Args` comes from `run.args` per tier run, not per invocation, so the plan must carry args per invocation. The existing `inv.partial` path, which records nothing, is the alternative.
- D8, concurrent `testmap` rewrite against a gate read (`coveragemap.go:61-72`): survives. Reads and writes are locked and atomic, and a failed read is a miss.

**Findings**

1. **MEDIUM, design, D2.** `testmap` keeps the dirty-file skip and the files-changed-under-tests check to the package's own directory. If another package's Go file is modified or untracked in the prime, the cross-package map records HEAD hashes for that package. Its block keys and coverage, though, come from the working-tree content that `go test -c` built and `KeyBlocks` read. The map then passes D5's freshness check at a base equal to HEAD while describing different code. Evidence: `internal/gatecli/testmap.go:409-417` (`dirtyGoFile` checks only `path.Dir(file) == dir`), `internal/gatecli/testmappackage.go:213-238` (`changedFiles` checks only `dir`), `internal/testcover/blockkey.go:89-96` (`KeyBlocks` reads the file from disk), `internal/gatecli/testmap.go:421-439` (`fileHashes` reads HEAD). Fix: D2 extends the skip and the post-run change check to every covered package's files, or records covered-package hashes from the files actually built.

2. **MEDIUM, design, D2/D5.** The format marker and the cross-package field survive a rewrite by an older binary. `WriteMap` keeps every key of the replaced record that `Map` does not declare. The maps directory is shared by every worktree of the hub on the machine. An older `lym gate testmap`, still deployed on another checkout or running mid-update, rewrites a map with own-package-only `Blocks` and `Tests`, but the new marker and the old covered-package file sets stay. D5 then reads a "new-format" map whose tests no longer hold cross-package blocks, selects nothing, and narrows wrongly. Evidence: `internal/testcover/coveragemap.go:66-72` ("It keeps each key of the replaced record that Map does not declare"), `internal/testcover/coveragemap.go:17-21` (one `Dir` shared by every worktree). Fix: tie the marker to a declared field that every writer rewrites, for example a marker holding the map's `BuiltAt` or `Commit`, read as a miss on mismatch, and state it in D2.

Outside my lens: the PATTERN-fabric-boundary constraint names "the path the impactset diff already uses" for reading base file content. `impactset` reads only names, through `git diff --name-status` in `internal/impactset/targets.go:109`. The existing content path is `fabricengine`'s `CodeFileAt`, through `goFileHashes` in `redundancy.go:350`.

Tool calls:
1. `sed`/`grep` over `internal/gatecli/redundancy.go:285-345` and `575-600`, plus a repo grep for map readers and scope-value readers: my context lacked the redundancy report's map read and kept-set build, and the full list of `ReadMap`/`WriteMap`/scope consumers.
