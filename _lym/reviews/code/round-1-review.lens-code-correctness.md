```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens over the functions, branches and error paths this diff adds or changes.

**What I ran, function by function**

- `testMapCmd`, `--max`: four inputs.
  - No flag gives `limit = math.MaxInt`.
  - `--max 0` and `--max -1` reach the `maxPackages < 1` refusal, which names the way forward.
  - `--max 3` gives 3.
  - Correct.
- `runTestMap` output: the `planned_head`, `queued`, `plan` and mismatch fields read straight from `run`. A refusal before HEAD is read gives an empty head. Correct.
- `mapPackages` setting `r.next`:
  - With i at the last job, next is empty.
  - With `measured+1 == maxPackages`, next is empty and nothing is built ahead.
  - With `maxPackages = MaxInt`, `measured+1` does not overflow.
  - After the loop, next is reset, so the audit builds nothing ahead.
  - Correct.
- `measureJob`:
  - Lock not held: the job is skipped and counted as queued.
  - Lock held: it is released by `defer` on every return.
  - Correct.
- `mapPackage` and `takeBuild`, five cases:
  - `LinkedPackages` fails: `linked` is nil. A live ctx goes to `writeFailed(build.err)`, and a cancelled ctx returns `ctx.Err()`.
  - `BuildBinary` fails: the dirty check comes first, then `writeFailed` or `ctx.Err()`.
  - A dirty linked file: the job is skipped as "before build", no ahead build starts, and the binary is removed by `defer`.
  - The ahead build belongs to another package because job i skipped before `takeBuild`: it is waited for, removed and rebuilt. The result is correct, but see finding 1.
  - Correct.
- `startBuild`:
  - `r.builds++` runs on the main goroutine only.
  - The fields written in the goroutine are read only after `<-done`.
  - `holderEnv` is set, since the build starts only after `holdSlot`.
  - Inside an inherited slot, `takeSlot` still sets `holderEnv` with a nil lease.
  - Correct.
- `measureTests`:
  - Whole job with N tests: `checked` is true, so there is one batch, then `endJob`, a single write, and `measure.Tests = N`.
  - Non-whole job, first batch writes no profile: `table` stays nil, the loop probes a kept entry (`probed`), then ends.
  - Non-whole job, no kept entry to probe: `break`, `runs` is empty, it returns nil and writes nothing. The plan already wrote the map.
  - Self-check failures are queued into a further batch, and `ranNames` excludes the tests that just ran.
  - A `runBatch` error goes to `ctx.Err()` or to `writeFailed(p.m)`, as the old key failure did.
  - Correct.
- `runBatch`:
  - `parallel = 1` or a single test: no re-run.
  - A failed or timed-out run beside others is replaced by one run alone, and a timeout is listed only from the final runs.
  - A reduce error from the alone run is propagated.
  - Correct.
- `runSideBySide`:
  - Two parallel workers and five tests: the feeder sends on the unbuffered `indexes`, then closes it, waits for the workers and closes `ended`. The main goroutine drains `ended` to the close, so there is no deadlock.
  - Each worker has its own profile path.
  - `runs[i]` is published through the `ended` send, which orders it before the main goroutine reads it.
  - After the first reduce error, later profiles are dropped and the error is returned.
  - A cancelled ctx returns `ctx.Err()` and discards the batch.
  - Correct.
- `reduce` and `newBlockTable`:
  - Nil profile: returns nil and keeps nothing.
  - A failed run's profile still sets `table`, which is valid, since every profile holds every block.
  - `profiled` is built only while `!checked`, so a whole job gets none and `selfCheck` never runs for it.
  - A `KeyBlocks` or `ProfiledFuncs` error is wrapped and returned.
  - Correct.
- `selfCheck`: `table.profiled` is non-nil whenever the function is called, since `checked` was false when the table was made. Correct.
- `fillMapTests`, with `table` nil:
  - A passing run without a profile has a nil `covered`, so `table.positions` is never touched and there is no nil dereference.
  - The blocks fall back to `m.Blocks`.
  - `entry.Funcs` is compacted.
  - Correct.
- `endJob`: unchanged logic, now called once per job. Correct.
- `close`: waits for the ahead build, then releases the slot and the temp directory. Correct, but see finding 1.
- `audit`: takes `r.pool.TryAcquire` only, and an already held slot returns true. Correct.
- `buildLym`, with `r.ctx` cancelled during the build: `lymBuilt` is reset and `ctx.Err()` returned. Correct.
- `ProfileByPackage`:
  - A root-module file `example.com/m/x.go` groups under `example.com/m`.
  - A nested package's file groups under the nested package.
  - It matches `PackageBlocks`.
  - Its only callers are the two in `reduce` and `newBlockTable` (`testmappackage.go:436`, `:467`).
  - Correct.
- `FuncIndex.CoveredFuncs` and `ProfiledFuncs` with the narrowed input: production calls them only from `testmappackage.go:436` and `:467`, both with `groups[importPath]`, so no caller still passes a whole profile. Correct.
- `gateslot.migrateCoverageScopes`:
  - A flow mapping with `scope: coverage` is rewritten to `diff`.
  - A second pass changes nothing.
  - A list-shaped entry is skipped, and `migrateGateLists` never emits `coverage`.
  - Correct.
- `decodeGates` with the retired value: the refusal names the way forward and mirrors the existing retired-list-shape refusal (`config.go:454`). Correct.
- `dobbyengine` config: `test_map_upkeep`, `test_map_every` and `test_map_batch` are in `RetiredKeys`, so a layer holding them loads unchanged. `MigrateLayer` and `configreg`'s `Migrate` are both gone. Correct.
- `orchcli.beamDown` after the `endTestMap` removal, and `fabriccli.liveUnsparedChild` without `RoleTestMap`: correct.

**Findings**

1. LOW, design, `internal/gatecli/testmappackage.go:236-249` (`takeBuild`) and `internal/gatecli/testmap.go` (`close`, `<-r.ahead.done`).
   An ahead build that will never be used is waited for in full instead of being cancelled. The build runs under `r.ctx` (`testmappackage.go:216-217`), so nothing ends it early.
   - When job i+1 skips before `takeBuild` (lock held, failed at HEAD, map moved, no mapped test), `takeBuild` for job i+2 blocks until job i+1's whole `go list` and build end, then removes it and starts its own build.
   - After a HEAD move or an error ends the walk, `close` blocks until the ahead build ends. That can take up to `redundancy_build_timeout_min` before the call exits.
   - The results are correct; only time is wasted.
2. LOW, consistency, `internal/gatecli/doc.go` (the line in the diff "since each process has its own environment and working directory").
   That reason is wrong for the working directory. `runTest` starts every run in the same `p.pkgDir` (`testmappackage.go`, `runner.RunAlone(p.bin, p.pkgDir, …)`).
   - Tests that do not call `t.Parallel` and write fixed paths in the package directory can interfere when run side by side.
   - Only a failure or timeout gets the re-run alone (`runBatch`). A run whose outcome or coverage is changed by another run but still passes is recorded as it ran.

Tool calls:
- One `grep` for the callers of `CoveredFuncs` and `ProfiledFuncs`, and for gateslot's load-time migration and refusal wording. My context held the callee change but not every caller of the narrowed input.
