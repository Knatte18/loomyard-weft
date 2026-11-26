```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each change in the diff to the board entry's goal points.

**Decisions** (decision record alone; the record has no `Added after Discussion` entries)
- G1: "Find why the reach map misses direct calls from tier1 tests, and fix the mapping."
- G2: "Where the map cannot see a call, have the caveat name the reason for that function."

**Per change**
- C1, the map run now runs tests marked unresolved and stores their verdict beside their coverage. This is `internal/gatecli/testmappackage.go`: the `mapPackage` filter, `packageRun.spawns`, `mappedRun.spawn`, `runTest` and `fillMapTests`. Traces to G1.
- C2, the new `MapTest.HoldsCoverage`, which `buildTable` now uses to read reach (`internal/testcover/coveragemap.go`, `internal/testreach/reader.go`). Traces to G1.
- C3, `MapFormatVersion` goes from 2 to 3. Forced by C1, since the shape of a stored entry changes.
- C4, `UnseenTest` and `Reader.UnseenTests` replace `SpawnMarked`, with `packageInfo.Unseen` and `indexVersion` 2 (`internal/testreach/reader.go`, `internal/testreach/index.go`). Traces to G2.
- C5, the reach caveats now name each cause and its tests (`internal/gatecli/reach.go`: `caveats`, `unmappedTierNames`, `unseenTestCaveats`, `unseenCause`, `namedTests`, the cause constants). Traces to G2.
  - The outcome caveat for failed, timed-out and skipped runs is new; before, no caveat covered those tests.
  - It still traces to G2, because a run that recorded no coverage is a call the map cannot see.
- C6, the docs: `internal/gatecli/doc.go`, `internal/testcover/doc.go`, `internal/testreach/doc.go` and `pattern/PATTERN-test-speed.md`. Forced by C1, C2, C4 and C5.
- C7, the tests: `TestGateTestMapLinkedPackages`, `TestReader`, `TestReaderIndex` and the new `TestUnseenTestCaveats`. Forced by C1, C2, C4 and C5.

No change is unrequested scope, and no commit in the range carries a `Pattern-Departure:` trailer.

**Per decision**
- G1 is built by C1, C2 and C3.
  - Tier1 tests marked out of process stay unrun on purpose, and the summary's design decisions give the reason.
  - G2 covers them through the `causeOutOfProcess` caveat, so this is no scope gap.
- G2 is built by C4 and C5, but only partly (finding 1).

**Findings**

1. MEDIUM, class `scope`, `internal/gatecli/reach.go:458`. G2 is built only for tests in the function's own package.
   - `caveats` calls `reader.UnseenTests(pkg)` with `pkg` set to the function's own package.
   - A test in another package can call the function and still be unseen: marked unresolved with no coverage in an old map, failed, timed out, skipped, or out of process. No caveat names the reason for any of those.
   - The one cross-package caveat is the stale-linker one, which only applies when `fn.New` is set (`reach.go:459-469`).
   - The board entry asks for the reason "where the map cannot see a call", with no limit to calls from the function's own package.
   - Evidence: `internal/gatecli/reach.go:449-471` and `internal/testreach/reader.go:260-261`, where `UnseenTests` returns the given package's own `Unseen` only.
   - Fix: also name the unseen tests of the packages whose maps link the function's package, or say that those packages' tests go unnamed.

One point for the handler, outside this lens: nothing reads `MapFormatVersion`, and `Fresh` ignores it. A fresh version-2 worktree map therefore keeps its never-run unresolved entries, so G1's fix does not reach an existing worktree until that package is mapped again.

Tool calls: none.
