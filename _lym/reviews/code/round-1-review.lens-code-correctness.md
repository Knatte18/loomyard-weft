```yaml
lens: code-correctness
outcome: reported
```

**Task:** the code-correctness lens over the diff's `internal/gatecli` map-run code, one concrete case per function or branch.

**Per-function list**

- `flagShapeRefusal` (`--map` branch): `--tier 1 --map` is refused. The message names `lym gate test --map` with targets, and `--tags` is also refused. Correct.
- `mapSideBySide`:
  - `Parallel 4, MemoryMB 256, testMemory 128MB` gives 2. Correct.
  - `MemoryMB` equal to `testMemory` gives 1. Correct.
  - `MemoryMB` unset follows the capped count. Correct.
  - `MemoryMB` below `testMemory` never reaches it, because `prepare` refuses first. Correct.
- `mapSlotAsk`: `{0:{Slots:1}, 1:{Slots:0}}` gives 0, meaning every slot. `{Slots:2}` alone gives 2. The empty map cannot reach it, because `prepare` refuses a hub with no mapped tier. Correct.
- `prepare`: the go-json refusal, the memory refusal (naming tier, both values and `lym config gate`), and the module-root refusal are all correct. A tier1 that fails `cfg.Tier` is silently left out, which is intended for a hub without tier1.
- `runBatch` (D5):
  - A failed or timed-out test beside others, in a queue of more than 1 with `sideBySide > 1`, reruns alone. The alone outcome replaces the first run, and a pass is listed in `passedAlone`.
  - A one-test queue or `sideBySide == 1` reruns nothing, which is correct because no sibling ran beside it.
  - `timedOut` is appended after the rerun, so only a timeout that stands is listed. Correct.
- `runSideBySide`:
  - A reduction error drains `ended` and drops the remaining profiles, so there is no deadlock.
  - Cancellation returns `ctx.Err()` and the batch is discarded.
  - A worker count of 0 would hang, but `clampParallelToMemory` always returns at least 1, so it cannot happen.
- `runTest`: a start error, a guard kill, a timeout, a skip and a non-pass each map to the right outcome. Correct.
- `fillMapTests`: a passing run with a nil profile has an empty `covered`, so `table.positions` is never dereferenced with a nil table. Correct.
- `mapPackage`:
  - A package with no mapped test, or only spawn-marked tests, is written after a single clean check. Correct.
  - A failed listing (`linked == nil`) calls `writeFailed(..., nil)`, which checks the whole module. Cancellation is checked first. Correct.
  - A failed build with a known linked set checks the linked trees before writing the failure. Correct.
  - A compiled build is checked, then tests run, then `endJob`, then the write check. All paths are guarded. Correct.
- `takeBuild` and `close`: a wasted ahead build for a package that needs no binary is cancelled and waited on. It costs time only. Correct.
- `writeFailed`: an existing map with tests keeps its entries and gains `Failure`, otherwise it gets `BuildFailure`. A dirty tree or moved HEAD writes nothing and lists the package unmeasured. Correct.
- `mapPackages`: a moved HEAD lists the current package plus every later package, ends the walk, and exits 1 through `unmeasured`. A dirty package continues. Correct.
- `runTestMap`: exit 1 comes from `failed`, `failedTests` or `unmeasured`. `passedAlone` alone gives 0. Cancellation gives 128+n. Correct.
- `takeSlot`: with a nil lease inside an inherited slot, `releaseHeldSlot` is a no-op. Correct.

**Findings**

1. **LOW, design**: `internal/gatecli/testmaptree.go:79` (`dirtyOwnedFile`) matches paths from `fabric.Status()` against package trees.
   - `Status()` merges both sides of the warp/weft pair, and weft entries are records-side paths (`internal/fabricengine/diff.go:130-149`).
   - A records-side path is compared against code-side package ownership. A module-root package (`dir "."`, which owns every non-nested path) would be reported dirty because of an unrelated records edit.
   - `dirtyModuleFile` (`testmaptree.go:102`) deliberately uses code-side changes only, so the two checks disagree.
   - Fix: filter `Status()` to `SideWarp` entries in `dirtyOwnedFile`, or call `CodeWorktreeChanges`.

No other defects found.

**Tool calls (3, in one message)**
1. `grep PrebuiltLymEnv` in `gateslot` and `gatecli`: the manifest showed no env-stripping code. I found `StripPrebuilt`, but did not confirm it is applied in map mode, and make no finding on it.
2. `fabricengine/diff.go` `Status` and `CodeWorktreeChanges`: the manifest did not show whether `Status` includes the records side. It does.
3. `redundantmutants.go` `memoryCappedParallel` and `clampParallelToMemory`: I needed to know whether the clamped count can be 0. It cannot.
