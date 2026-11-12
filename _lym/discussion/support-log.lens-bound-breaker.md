```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked the record's claims and bounds to find counterexamples.

**Findings**

1. **BLOCKING: D4 and D7. Uncommitted maps make the weft dirty, and the tier-2 preflight then refuses.**
   - D4 has a pass write maps through its whole run and commit only at the end. A killed pass leaves its maps uncommitted until the next pass.
   - `preflight.Check`'s worktree-clean check counts everything under `_lym` except the run-records root and the loom friction dirs (`runStateExemptRels` in `internal/preflight/preflight.go`). So `_lym/testcover/` counts as dirty.
   - Counterexample 1: in a task worktree, someone runs a pass by hand (D7) while the task's run is live. The run's orchestrator preflight fails worktree-clean until the pass commits, which can take tens of minutes.
   - Counterexample 2: in the prime, a pass is killed (it yields to gates, or the machine sleeps). Every tier-2 orchestrator in the prime then refuses until the next pass, and with D8 the upkeep is off by default, so that pass may never come.
   - The record says nothing about this. The fix is either to add `_lym/testcover/` to `runStateExemptRels` (a design decision with a bound) or to commit per package.

2. **MEDIUM: D5. "Two machines never commit one branch's maps" fails on a hub with no prime owner recorded.**
   - `PATTERN-owner-machine` says the root of a hub with no `prime_owner` recorded reads as owned by no other machine.
   - Counterexample: two machines share such a hub and both have `test_map_upkeep` on. Both prime dobbies start passes and both commit the prime's weft maps. The weft pull's replay then meets a conflict on the same `map.json`.
   - The guard holds only once an owner is recorded. The record should state that bound or refuse when no owner is recorded.

3. **LOW: D4's bound "the commit covers only `_lym/testcover/`" holds for the pass's own commit, but other committers also sweep the directory up.**
   - Teardown's pending-records commit uses the record pathspec, and the `lym fabric` weft commit uses routing names, which are the whole `_lym` (`internal/fabriccli/weft_verbs.go`).
   - Either one commits half-written maps that a pass left behind. So "the next pass's commit takes them" is not the only path.
   - This is harmless for correctness, but the record's claim is not exact.

4. **LOW: D1 and D2. A pair created before the change has no `_lym/testcover/`, and the record does not mention it.**
   - Its weft forked before D9's import, so after the upgrade its gate reads `MissNoMap` for every package and runs whole packages.
   - That is correct, but slower until the pair is torn down.

**Claims attacked**
- D2, the fork drops only run records: held. `forkrecords.go` drops only `shedrun.RunsRootRel`.
- D2, a fork adds no new objects: held.
- D6, the selector diffs from the map's own commit, so an unmoved map stays valid: held. The selector compares the recorded `Files`, `OtherFiles` and `Linked` hashes with the target's, and an unchanged map keeps `Format.Commit` equal to `Commit`, so `MissUnmarked` is not triggered.
- D6 bound, the map's commit stays reachable: held for the prime's history and for a task branch forked from the parent.
- D4 and D7, the exit record and locks stay never-tracked: held.
- D4 and D7, a pass is safe beside a live run: broken (finding 1).
- D5, a single committer per branch: broken when no owner is recorded (finding 2).
- D4, a killed pass's maps are committed by the next pass: held, but not the only path (finding 3).
- D9, a second run imports nothing: held. A failed removal after the commit leaves a stale bolt directory, but the prime's maps then exist, so nothing is re-imported.
- D8, no reinterpretation needed: held. It is a default change, and a tuned base value is kept.

**Tool calls**
- Grepped `forkrecords.go` and `testselect.go`, to check D2's drop scope and D6's selector basis, which the context only summarised.
- Grepped the `ScopedPathspec` callers and Remove, to find other committers of `_lym` (finding 3).
- Grepped `internal/preflight` for clean and dirty handling, to test whether uncommitted maps break the tier-2 clean check (finding 1).
- Read `runStateExemptRels` and the preflight doc, to confirm which `_lym` paths are exempt.
