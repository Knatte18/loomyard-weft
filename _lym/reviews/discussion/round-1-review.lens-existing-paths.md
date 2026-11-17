```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced every state, file and invariant the record changes against the code that already reads or writes it.

**Paths traced, per decision**

- **D1, detached keying.** `testRequest.words` → `callKeyDir`/`detachKeyDir` (detachfront.go:343-391): survives, provided `--map` is added to `words()` as D1 says.
- **D1, card fixer.** `handsOff` (cardfixer.go:79-92) needs `--gate`, which map mode refuses, so the fixer never fires on a red map call: survives.
- **D1, join check.** `joinable` compares tree identity, and a map run leaves the tree unchanged: survives.
- **D3/D4, shared helpers.** `prebuildLym`, `renderTierEnv`, `cappedParallel` and `mappedTiers` are shared with `redundancy.go`, `redundancytests.go` and `redundantmutants.go`. They are not only-testmap code, so D9's removal must keep them. `mappedTiers` sits in `testmap.go:46-47`, a file D9 deletes. Survives, as plan detail.
- **D6, today's dirty-file checks.** `mapPackage` checks after the build is taken up (testmappackage.go:167-178), and `endJob` checks after the tests (:489-512). The bound `doc.go` states today is dropped. **Breaks**, see F2.
- **D7, map entries after a failed build.** `writeFailed` keeps the commit and entries and sets `Failure`. Readers: the redundancy step judges by the entries (redundancy.go:280-311), and `lym gate redundant` prints `map_build_failure` (redundant.go:214). Survives.
- **D7, records.** No run, green or failure record is written. Readers are `lym gate report` and the tier carry; the ordering reader goes. Survives.
- **D7, no `FailedAt` skip.** `Map.FailedAt` is read only by the planner and audit (testmapplan.go:68, testmapaudit.go:37), which both go. A whole-rebuild call now rebuilds a failing package every time; that matches D2. Survives.
- **D8, where maps are read.** The location comes from `hubgeom.IsPrime` (testmap.go:323-330). The redundancy step reads only the hub's maps (redundancy.go:286), and `lym gate redundant` reads `coverageMapsDir` (redundant.go:156). Survives (the hub-only report step is already rejected in the log).
- **D8, `EnvironmentStamp`/`Selector`.** `NewSelector` and `stampDifferences` are read only by the planner (testselect.go:247). With planning gone, the stamp is written but no code path compares it. Survives (the marker is still written).
- **D9, `lym gate redundant` refusals.** All three refusals share one `rebuild` way-forward (redundant.go:162-174): unreadable map, missing map, build-failure-only map. D9 covers only the missing-map one. **Breaks**, see F1.
- **D9, `lym gate redundant` help text.** The Long text (redundant.go:104) and the doc comment (:129) name the verb. They are live text, but listing them is call-site enumeration, so not filed.
- **D9, `prebuild.go:1` header.** Names the verb; enumeration, not filed.
- **D7/Constraints, slot-holder invariant.** `PATTERN-gate-slot` allows a gate build or test only "in one of two places": a tier run, or a one-slot non-tier holder. **Breaks**, see F3.

**Findings**

F1 — LOW — design — D9 and its acceptance criterion
- **Summary:** D9 retargets only the missing-map refusal of `lym gate redundant`. The unreadable-map and "records only a failed build" refusals use the same `rebuild` text, which names `lym gate testmap`.
- **Impact:** once the verb goes, those two refusals point to an unknown command, which breaks `PATTERN-refusal-way-forward`. No criterion reaches them.
- **Evidence:** internal/gatecli/redundant.go:162-174.

F2 — MEDIUM — design — D6
- **Summary:** today's code and `gatecli/doc.go` state a bound for the dirty-tree guard: an edit made after one status read and reverted before the next goes unseen, so the map holds coverage of edited code under committed hashes. D6 claims a mid-run edit cannot reach a map and states no such bound.
- **Impact:** the hole still exists under D6's two checks. A file can be edited during the coverage build, compiled in, and reverted before the pre-write check. The guard is bypassable and its bound is not stated.
- **Evidence:** internal/gatecli/doc.go:1133-1134; testmappackage.go:167-178 and 489-497.

F3 — LOW — consistency — D7 and the `PATTERN-gate-slot` constraint
- **Summary:** the constraint says "a map run is a gate test run, so it holds slots", and D7 has it hold "the larger of tier 0's and tier 1's slot asks". The background's slot rule allows only a tier run (the tier's command, with its config) or a one-slot non-tier holder that sets `-p` from tier0's parallel.
- **Impact:** the map run is neither. The record removes the testmap holder from the list but changes nothing in `pattern/PATTERN-gate-slot.md`. Its Scope covers only the test-speed and daemon-wakeups backgrounds.
- **Evidence:** pattern/PATTERN-gate-slot.md:12-16; internal/gateslot/doc.go:64-70.

Tool calls:
1. `grep` for `FailedAt`/`Format`/`EnvironmentStamp` across internal and cmd: to find readers of the map fields D7 and D8 touch. Too broad, superseded by call 2.
2. `grep` in gatecli, testcover and cmd/lym for map-field readers, `ReadMap` and `IsPrime`: to find every reader of the map state.
3. `grep` for `Selector`/`PlanUpkeep` callers, and reads of redundancy.go:280-330 and redundant.go:150-180: to check whether consumers still read the maps correctly; not in the manifest.
4. `git grep` with a pathspec that failed (`:!_lym`): wasted call.
5. `git grep` for testmap mentions outside the listed files: to find live paths naming the verb, which led to the redundant.go refusals.
6. `grep` for helpers shared between the redundancy and testmap code, and a read of redundant.go:98-110: to check what D9's removal would reach.
