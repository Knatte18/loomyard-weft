```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the diff's changed functions and error paths, read-only.

**Cases run**

| Function / branch | Case | Result |
|---|---|---|
| `flagShapeRefusal` map block | each refused partner; `--map --tags`; `--map --all` | correct; later shape checks still apply |
| `runTest` map hand-off | `mapMode` request | routes correctly, but skips the pre-tier ceiling (finding 3) |
| `words()` | map and plain call over `./pkg` | correct; keys differ |
| `runTestMap` | cwd error, location or prepare refusal, `computeScope` error, cancel in walk, exit-1 causes | correct |
| `prepare` | vstest tier, `memory_mb` below one cap, no tier1, bad module dir | correct; `Parallel` ≥ 1 and the test cap ≥ 1 are validated, so no zero-worker hang or division by zero |
| `packages` / `leftOutReason` | map exists, untracked package, scan error, named but left out | correct |
| `scopePackages`, `modulePackagesAt`, `ellipsisRoot` | `./...`, `../...`, empty share, inert-only share, `moduleRel` of `.` and `backend`, card naming an uncreated package | correct |
| `mapPackages` | zero packages; HEAD moved at package i | correct; current and later packages each listed once |
| `readHead` / `checkClean` | HEAD moved, nil trees, status failure | errors propagate; linked-tree path reads records-side entries too (finding 2) |
| `mapPackage` | no tests, all spawn-marked, listing fails, build fails, cancel, dirty after build, stale ahead build | correct |
| `measureTests` | empty tier batch, undeclared tier, reduce error, cancel, dirty at end, write skipped | correct |
| `runBatch` / `runSideBySide` | one test, count 1, fail beside others then pass alone, cancel during rerun | correct |
| `runTest` (map) | spawn error, guard kill, timeout, skip | outcomes correct; log text stale (finding 1) |
| `mapSideBySide`, `mapSlotAsk` | `memory_mb` equal to the cap; a tier with `slots: all` | correct |
| `takeSlot` / `releaseHeldSlot` | inherited slot, count 0 or 1, cancelled acquire | correct |
| `write` / `writeFailed` | check fails, kept map with entries, `WriteMap` error | correct |
| `endJob`, `skip`, `close` | early end with a pending ahead build | correct |
| `runRedundant` refusals | missing map, stale Go files, package `.` | correct |
| `funcsHolding` after the parameter removal | zero-count block | correct |

**Findings**

1. **LOW, design** — `internal/gatecli/testmappackage.go`, `testMapRun.runTest`.
   Both `logger.Warn` messages still open `gate testmap:` ("a test that could not run…", "a test the memory guard killed…").
   A `lym gate test --map` run therefore logs failures under the retired verb, which now answers as an unknown command.
   Evidence: `testmappackage.go`, the `switch` at the end of `runTest`.

2. **LOW, design** — `internal/gatecli/testmaptree.go` `dirtyOwnedFile`, reached from `testmap.go` `checkClean` for non-nil trees.
   It reads `Fabric.Status`, which merges records-side (weft) paths, relative to the records checkout, into the code paths.
   A module-root package `.` at the worktree root, or a package directory that contains the anchor's `_lym`, therefore "owns" an uncommitted run record.
   It is then listed unmeasured, with a way forward naming a file that is not part of the code checkout, and the call exits 1.
   The nil-trees path (`dirtyModuleFile`) reads code changes only, so the two checks disagree.
   The helper predates this diff; the diff now applies it before every write, no-build packages included.
   Evidence: `fabricengine/diff.go` `Status`; `testmaptree.go` `packageTree.owns` and `dirtyOwnedFile`.
   This follows from reading the code; I did not run it.

3. **LOW, design** — `internal/gatecli/run.go` `runTest` map branch, and `testmap.go` `runTestMap`.
   The branch returns before `preCtx` is built, so `computeScope` runs under a context with no deadline.
   Under the go scoper a computed, `--base` or `--card --main-packages` scope runs `go list`, and `computeScope`'s doc relies on `ctx` to kill a stalled one.
   A stall there hangs a `--map` call (or its detached runner) until a signal, where a plain call is bounded by `preTierCeiling`.
   D1's stated bound lists the tiers' timeouts, not this ceiling.
   Evidence: `run.go`, the `if request.mapMode` return above `context.WithTimeout(runCtx, request.preTierCeiling)`; the `computeScope` doc in `scope.go`.

**Tool calls**
1. Grep of `scope.go` and `dotnetscope.go` for the request fields they read, tier validation in `gateslot`, and `noneScope`: the manifest lacked which `testRequest` fields scope derivation needs, and whether `Parallel`, `Slots` or the test cap can be zero.
2. `scope.go` share construction and `tiers.go` defaults and `tier_parallel` validation: the manifest lacked how card and diff shares are built, and the default `Parallel` and `Slots`.
