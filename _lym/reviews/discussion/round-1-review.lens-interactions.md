```yaml
lens: interactions
outcome: reported
```

Task: interactions lens. I checked each pair of decisions that act on the same state, and each acceptance criterion and constraint against the decisions it touches.

**Pairs checked**

- D1 base per tier × D5 freshness at the tier's base: **broken** (finding 2).
- D2 staleness at HEAD × D5 freshness at the tier's base: **broken** (finding 1).
- D2 own-package `Files` beside the covered-package field × D5 "for a covered package": **broken** (finding 3).
- D2 format marker × Constraint PATTERN-tolerant-records (`WriteMap` keeps undeclared keys): **broken** (finding 4).
- Constraint PATTERN-fabric-boundary × D4 base file content: **broken** (finding 5).
- D3 acceptance criterion "nothing outside it but D6's same-name overlap" × D6 over-budget whole run: **broken** (finding 6).
- D7 labels × D3 binary-test whole packages and D6 over-budget whole runs: **broken** (finding 7).
- D5 acceptance criterion "a new file in a covered package" × D4/D5 new-file handling: **broken** (finding 8).
- D1 "never adds a package `diff` would not run" × D3 union and D6 overlap: held. Both stay inside packages already in the `diff` set.
- D1 explicit targets never narrowed × D6 failed-test rerun: held. `FailedTestsCall` renders explicit packages plus `-run`, which D1 leaves un-narrowed.
- D1 watch skips and guard tests × D3 guard union and D6 `-run`: held. The skip still applies, and `targetsFor` already drops unwatched tests from Guards.
- D1 guard step × D3 narrowed-package guards: held. Importers sit inside the set, so D3 owns their guards and the guard step does not.
- D2 spawn verdict × D3 always-run out-of-process tests: held. Tests that failed or timed out alone are absent from the map, and D3 runs them.
- D2 tests covering only other packages × D3 unrecorded tests: held.
- D2 failed-build map with marker × D5 build-failure miss: held.
- D2 cross-package keys carrying an import path × D4 match by file and function: held.
- D2 redundancy own-package judgment × D2 cross-package `Blocks`: held, provided the rendered key carries the import path, which D2 states.
- D4 changed or deleted functions × D5 non-body, new-method and `init` misses: held. A signature change falls under D5's non-body rule.
- D4 deleted file × D5 file-set check at the base: held. The file still exists at the base.
- D5 unmapped tier × D3 per-tier union: held.
- D5 non-Go file × D1 `inert_paths`: held.
- D6 selected green entry with args × the base lookup `newestWholeAncestor`: held. Entries with args are excluded.
- D6 `--stop-on-fail` and rerun × D3 union: held.
- D8 `ReadMap` under its lock × D2 atomic `WriteMap`: held.
- D8 × Constraint PATTERN-gate-slot: held.
- D1 × Constraint PATTERN-config-layers: held.

**Findings**

1. **BLOCKING · design · D2 × D5.** D2's staleness rule keeps rebuilding maps at prime HEAD whenever any covered package drifts. D5 then reads every map built after a worktree's base as a miss, so D2 actively invalidates maps for every running worktree.
   - Evidence: `internal/gatecli/testmap.go` `stale` compares at `head`, and testmap runs only in the prime (`testMapLocation`).
   - `git log` on main from 2026-11-04 to 2026-11-11: 52 of 163 commits touch `internal/configreg`, `internal/loomengine`, `internal/gateslot`, `internal/fabricengine` or `internal/state`. Those are exactly the widely imported packages the board names.
   - Open risks names the effect as "hit rate" but leaves the fact unmeasured, and no Decision reconciles D2 with D5.
   - Fix direction: diff D4 from each map's recorded `Commit` instead of the tier base. A function changed since the map was built is then selected, and the hash check against the base goes away. Otherwise, record the measured hit rate.

2. **MEDIUM · design · D1 × D5.** D1 keeps the impactset base per tier, which is the newest whole-green ancestor and can be a branch commit. D5 requires the map's hashes to match the files at that base, but maps are only ever built from main in the prime. For any package the branch changed before that green commit, every importer misses.
   - Example: the hub's `webster` runs tiers 0 to 2 with scope `all`. If tiers 0 and 1 pass and tier 2 fails, whole entries are recorded at a branch commit. `webster-fix`, one of D1's three default keys, then diffs from that commit and reverts to plain `diff` for every importer of a package the branch touched.
   - Evidence: `internal/gatecli/green.go:202` (`newestWholeAncestor`), `internal/gatecli/scope.go:390-407`, and `_bolt/_lym/config/gate.yaml` lines 45-46.

3. **MEDIUM · design · D2 × D5.** D2 defines the new field "per covered package other than its own" and keeps own-package `Files` separate. D5's hash rule reads "for a covered package", which in D2's vocabulary excludes the importing package itself.
   - Failure case: a test of B whose body changed on main between the map build and the base keeps its stale blocks, because D3 runs only tests absent from the map. It is then not selected even though it now reaches the changed function. That silently narrows the scope, against "widen, never narrow".
   - Fix: D5 states that the importing package's own `Files`, or the `MapTest.FuncHash` of each test, must match at the base.
   - Evidence: `internal/testcover/coveragemap.go` `Map.Files` and `MapTest.FuncHash`; decision-record.md lines 49 and 92.

4. **MEDIUM · design · D2 marker × Constraint PATTERN-tolerant-records.** `WriteMap` keeps every key of the replaced record that `Map` does not declare.
   - Failure case: an older lym binary rebuilds a map with own-package coverpkg and carries over the new marker and covered-package field from the replaced record. The gate reads it as a new-shape map with no cross-package blocks, so D4 selects nothing for importers and D5 sees no miss.
   - The marker does not stop an older writer.
   - Fix direction: bind the marker to the build, for example a marker holding the `Commit` or `BuiltAt` it was written with, checked against the top-level value.
   - Evidence: `internal/testcover/coveragemap.go` (`WriteMap` doc: "It keeps each key of the replaced record that Map does not declare"), decision-record.md line 131.

5. **MEDIUM · design · Constraint PATTERN-fabric-boundary × D4.** The constraint says D4's base file content is read "through the path the impactset diff already uses". That path only lists changed names (`gitexec.Run` `diff --name-status` and `ls-files`) and never reads file content. D4 therefore has no such path, and an implementer must add a new git read anyway.
   - Fix: name the existing content reader that testmap already uses, `Fabric.CodeFileAt`.
   - Evidence: `internal/impactset/targets.go:109,118`, `internal/fabricengine/codereads.go:27`, `internal/gatecli/testmap.go` `fileHashes`.

6. **LOW · consistency · D3 acceptance criterion × D6.** "An importing package runs … nothing outside it but D6's same-name overlap" contradicts D6, where a pattern over the command budget runs the selected packages whole.
   - Evidence: decision-record.md lines 111 and 164.

7. **LOW · consistency · D7 × D3/D6.** D7's four labels have none for a package run whole because its tests use the `lym` binary (D3), nor for a selected package run whole past the command budget (D6). D7's criterion requires every package to carry one of the four labels.
   - Evidence: decision-record.md lines 65, 111 and 119-120.

8. **LOW · consistency · D5 acceptance criterion × D4/D5.** "A new file in a covered package" as a miss case maps only to the file-set check at the base, that is, a file added between the map build and the base. Read as a file the diff adds, it contradicts D4/D5: a new file with only plain functions is no miss.
   - Evidence: decision-record.md lines 92, 97, 102 and 169.

Tool calls: none; my context already held everything this lens needed.
