```yaml
lens: code-correctness
outcome: reported
```

Task: review the diff for code-correctness. For each changed function or branch I ran one concrete case and checked it.

**Per-function list**

- `runTestMap`, `--max 0`: refused with a way forward. Correct.
- `testMapLocation`, prime and non-prime: `isPrimeWorktree` is true when `.git` is a directory, and for a linked worktree true only when its gitdir's parent is not `worktrees`. Both cases correct.
- `testMapRun.prepare`, hub declaring only tier1: `moduleDir` comes from the lowest declared tier, so it is correct.
- `testMapRun.prepare`, no tier declared: refuses. Correct.
- `testMapRun.packages`, named `./alpha`, `.` and a missing directory: each handled correctly. A named package with no tracked Go file is dropped silently, with no report (see F3).
- `testMapRun.mapPackages`, dirty and locked skips: both are decided before any build and are not counted against the cap. Correct.
- `testMapRun.mapPackages`, `--max 1` with 3 stale packages: 1 is built and 2 counted as unbuilt. Correct.
- `testMapRun.stale`, a failure map at a different HEAD: stale. Correct.
- `testMapRun.mapPackage`, a package with no mapped test: gets a map with no tests and no build, and counts against the cap. Correct.
- `testMapRun.mapPackage`, a failed build: gets a failure map that is stale again after a HEAD move. Correct.
- `testMapRun.mapPackage`, files changed mid-package: writes no map and reports a skip that counts against the cap. Correct.
- `testMapRun.mapPackage`, a HEAD move with unchanged hashes: keeps the map recorded with the pre-package HEAD. Correct.
- `testMapRun.holdSlot`, with and without an inherited slot, and `close` on every path: correct. The lease is nil when inherited, `defer run.close()` releases the slot and removes the temp directory, and the lock is released on each path.
- `testMapRun.buildLym`: bounded by a timeout, and the result is memoized, including a failure. Timeout versus other error is classified correctly. A call cancelled by a signal is misrecorded (see F2).
- `testMapRun.runTest`, timed out, failed, skipped and a tier with no declaration: all left out of the map. Only a timeout is listed.
- `testMapRun.changedFiles`: an added or removed file counts as changed. Correct.
- `fillMapTests`, zero passing runs: gives a map with no tests and no failure. Per D5 that is acceptable.
- `Toolchain.command`, `RunAlone` and `goOutput`: on timeout the process group is killed and the wait is bounded by `Grace`. `ErrWaitDelay` is mapped to nil. Correct.
- `testMapRun.close` (exit record): written on every exit after the location check, including `prepare` refusals and mid-walk errors (see F4).
- `gateEnv` refactor in `run.go`: it now clones `base` instead of mutating `os.Environ()`. Same behaviour.
- `WriteExitRecord` and `ReadExitRecord`: correct.
- `ScanTestFuncs` and `isTestFuncName`: correct.
- `CodeDirAt` and `CodeFileAt`: correct.

**Findings**

- **F1 — MEDIUM, design.** `internal/testcover/runner.go:60-62,162-166`, with `internal/gatecli/testmappackage.go:59-60`: the coverage build runs with no working directory.
  - `Toolchain.BuildBinary` and `goOutput` never set `cmd.Dir`, and the `Toolchain` built in `mapPackage` carries no module directory.
  - So `go test -c <importPath>` runs in the process's cwd, not the module root `prepare` resolved.
  - It works only when the invoking cwd is inside the module. If the tier `dir` is a subdirectory such as `sub`, which `moduleRel` explicitly supports, and the verb runs from the worktree root, `go` finds no main module. Every package then gets a failure map recording a build failure.
  - The integration test hides this because its stand-in `go` ignores the working directory.
  - Fix: give `Toolchain` a `Dir` field set from `r.moduleDir`, or pass `-C`.
- **F2 — LOW, design.** `internal/testcover/runner.go:192`, `internal/gatecli/testmappackage.go:139-157`: the call's context never reaches a build or a test.
  - `command` uses `context.Background()`, so a terminating signal does not stop a running coverage build or test. The call waits until the process exits or hits its timeout.
  - A signal during the `lym` build is recorded as a build failure (`lym build: <err>`) in the package's map. That map is then retried only after a HEAD move.
  - Fix: pass the call's context into `Toolchain`. In `buildLym`, return the cancellation instead of writing a failure map when `r.ctx.Err() != nil`.
- **F3 — LOW, design.** `internal/gatecli/testmap.go:370-376`: a named package with no Go file tracked at HEAD, and not dirty, is dropped silently. D5 requires that skipped packages be listed with a cause, and the package neither appears nor counts. Fix: report a skip with a cause when `named` and `len(files) == 0`.
- **F4 — LOW, design.** `internal/gatecli/testmap.go:474-478`: `close` writes the exit record even when the walk ended on an error or after a `prepare` refusal. `Unbuilt` then holds a partial count, often 0, so a reader would take the cap as not binding. Fix: write the record only after the walk completes, or on an error path write a record the reader can tell from a clean zero.
- **F5 — LOW, design.** `internal/gatecli/prebuild.go:63-66`: the `lym` build uses `CombinedOutput` with `ConfigureGroupKill` but no `WaitDelay`.
  - A descendant that left the process group and still holds the output pipe stalls `prebuildLym` past `redundancy_build_timeout_min`.
  - D5 bounds the wait after a kill only for tests, and `goOutput` and `RunAlone` are bounded by `Grace`. The `lym` build is the one unbounded path.
  - Fix: set `cmd.WaitDelay` to the grace.

**Tool calls**
1. Bash (`cat internal/proc/procgroup.go`, `grep isPrimeWorktree`, `grep resolveRunDir`): the bodies of `ConfigureGroupKill`, `isPrimeWorktree` and `resolveRunDir` were not in my context. I used them to check `IsPrime`, the group kill and the module directory resolution.
