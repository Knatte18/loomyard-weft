```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end against the code in context and checked the end state against the decision's purpose.

**Decisions walked**
- **D1:** case `lym gate test --map internal/gatecli` as a detached agent call. `testRequest.words` gains `--map`, so the key differs from a plain call's and the runner replays map mode. The fixer stays off because `--gate` is refused (`handsOff` needs a `--gate` key). Purpose met.
- **D2:** case: a map run over `internal/gatecli` whose map holds an entry for a deleted test. A whole rebuild starts from an empty map, so the entry is gone. Purpose met, but the one-ahead build collides with D6's pre-build check (finding 2).
- **D3:** case: a tier 1 test that reaches `lym` through `lymbin`. The spawn scan marks it out of process, so it is never run, and the prebuilt `lym` reaches no test. Purpose not met for the prebuilt build (finding 1).
- **D4:** case: tier 1 with `memory_mb: 1024` and the template's `redundancy_test_memory_mb: 2048`. The count works out to 0, or to 1 under today's floor. Purpose not met (finding 3). Case: a package holding both tier 0 and tier 1 tests: the run order is unspecified (finding 4).
- **D5:** case: a test killed by the guard's pressure trigger beside others. The guard kill is recorded as a failure today, so it reruns alone, and the pass stands and is listed. Purpose met.
- **D6:** case: the next package's build starts one ahead while the current package's tests run. The linked set the check needs only exists after the build's own listing step. Purpose not met before the build (finding 2). Case: a dirty file in a nested package marks its parent unmeasured (finding 5).
- **D7:** case: a test fails alone. The call exits non-zero, writes the map with the failed entry, writes no run record and subtracts no green tier. Purpose met.
- **D8:** case: a detached map call from a task worktree. The runner resolves the same worktree and writes the anchor `.lym` maps. Purpose met.
- **D9:** case: `lym gate redundant <pkg>` with no map. It names `lym gate test --map <pkg>`, which writes the map that `redundant` reads from the same worktree. Purpose met.

**Findings**

1. **MEDIUM, design. D3 (decision-record.md:66, 68) and D3's acceptance criterion (decision-record.md:179).**
   The prebuilt `lym` that D3 builds once for tier 1 is never reached by any run test, so its criterion cannot be met.
   - The only reader of `LYM_PREBUILT_LYM` is `internal/testkit/lymbin` (`internal/testkit/lymbin/lymbin.go:50`).
   - The spawn scan marks every test that references `lymbin`, directly or through a followed call, as out of process, and an unresolved call as unresolved (`internal/testcover/spawnscan.go:170,355`; `pattern/PATTERN-test-speed.md:278-280`).
   - D3 runs neither kind of test (decision-record.md:68).
   - So the build costs time and reaches nothing, and the criterion "a tier 1 test of a map run reaches the `lym` the run built once" describes no runnable test.
   - Fix: drop the prebuilt-`lym` build and its criterion, or state why a test that runs could reach it.

2. **MEDIUM, design. D6 against D2 (decision-record.md:55-56, 98-99).**
   D6's check before the build needs data that only the build itself produces.
   - D6 checks the directories of "any in-module package it links" before the coverage build.
   - The linked set comes from `LinkedPackages`, which runs inside the build's own goroutine (`internal/gatecli/testmappackage.go:227`).
   - Today's code therefore checks only after the build is taken up (testmappackage.go:172).
   - With D2's one-ahead build, the next package's build starts during the current package's tests with no linked set known.
   - The record does not say whether the check before the build covers the package's own directory only, or runs after the listing step, before compilation.
   - This leaves Plan to work out a fact the design rests on.

3. **MEDIUM, design. D4 (decision-record.md:78-79).**
   The `memory_mb` clamp breaks when a tier's `memory_mb` is below `redundancy_test_memory_mb`.
   - Example: `memory_mb: 1024` against the template's 2048 (`internal/gateslot/template.yaml:7`). The division `memory_mb / per-test cap` gives 0.
   - The existing clamp, `clampParallelToMemory`, guarantees "One run always goes" (`internal/gatecli/redundantmutants.go:539`).
     Floored at 1, a single 2048 MB test exceeds the tier's 1024 MB.
   - Left at 0, `runSideBySide` starts `min(0, n)` workers and its feeder blocks forever on `indexes <- i` (testmappackage.go:386-399).
   - Either way, the stated purpose ("the tier's cap bounds the run's test processes together") is not met, and the record states no rule for this case.

4. **LOW, consistency. D4 (decision-record.md:75).**
   "A tier's tests run side by side up to that tier's `parallel`" does not say whether a package's tier 0 and tier 1 tests run at the same time or one tier after the other.
   - At the same time, a package runs `p0 + p1` processes: 8 on the `_bolt` hub, where each tier's `parallel` is 4.
   - That is within no single tier's `parallel`, and it is bounded only by D7's slot count.
   - Today's code runs one side-by-side count, taken from tier 0's parallel (testmappackage.go:604-608).

5. **LOW, design. D6 (decision-record.md:99).**
   "Any file under the package's directory" covers nested packages' files.
   - Example: a dirty file in `internal/testkit/tiergate/` would mark a package at `internal/testkit` unmeasured even though it does not own that file.
   - Today a package's files stop at the next package directory (`internal/gatecli/doc.go:1071-1072`, used through `dirtyOwnedFile` at testmappackage.go:172).
   - The record should say "files the package or a linked package owns".

**Tool calls**
- Grep of `spawnscan.go` and the `PREBUILT_LYM` readers: my context did not show which references the spawn scan marks or who reads the prebuilt variable.
- Grep for `cappedParallel`: no match.
- Grep for `cappedParallel(` call sites, which reached `memoryCappedParallel` and `clampParallelToMemory`: my context lacked the clamp's floor behaviour.
