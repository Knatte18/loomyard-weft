```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. For each state the record changes, I traced the existing code paths that write, reset or read it.

**Paths traced, by decision**

- **D1:**
  - The gate's coverage scope (`internal/gatecli/run.go:83`), `redundancy.go:282`, `redundant.go:94`, the testmap verb (`testmap.go:262`) and the dobby's exit-record read (`internal/beamcli/dobbydeps.go:244`) all pass `fabricengine.BoltDir`. D1 moves them all. Survives.
- **D2:**
  - `internal/fabricengine/forkrecords.go:48,59` takes only the run-records root out of the index, so `_lym/testcover/` is carried to the child. Survives.
- **D4:**
  - A teardown's pending-records commit sweeps in leftover maps. The record names this. Survives.
  - The weft pull's dirty stop (`internal/fabricengine/pull.go:406-415,440-445`) refuses a weft move while tracked uncommitted paths stand. **Breaks:** the record is silent, see F1.
- **D5:**
  - Preflight's `runStateExemptRels` (`internal/preflight/preflight.go:26`) gains the subtree. Survives.
  - The fabric weft pull is a second consumer of the same dirt, and the exemption does not reach it. **Breaks** (F1).
- **D6:**
  - `Selector.environmentMiss` and `Selector.commitMiss` (`internal/testcover/testselect.go:243-273`), and the `HubEnvDigest` over `envconfig` `Vars` (`testmappackage.go:818`, `envconfig/config.go:56-65,97`). Read right. Survives.
- **D7:**
  - The audit filter (`testmapaudit.go:38`) and the package-job guard (`testmappackage.go:82`) are named. Survives.
  - `PlanUpkeep`'s `Changed` rule (`upkeep.go:77`) is the line D7 changes. Survives.
  - The dobby's work-left check (`dobbyengine/testmap.go:89`) reads `PlannedHead`. Survives.
  - The redundancy step's `MapAge` and `mapDrift` (`redundancy.go:315-318,346-364`) read a map's `BuiltAt` and `Files` as a measure of staleness. **Misreads** under D7, see F3.
  - `lym gate redundant` reports `map_commit` and `map_built_at` (`redundant.go` answer map). The answer is informational only and drives no decision. Survives.
- **D9:**
  - The dobby's `testMapRunning` reads `HubOwner` only (`dobbydeps.go:619-624`). Survives D9's bound.
  - The registration lock sits in the hub's scratch directory (`testmap.go:326`). Survives.
  - Only a `--upkeep` call registers (`testmap.go:252-256`); `--upkeep` is hidden and refuses named packages (`testmap.go:135,238`). **Breaks** D9's teardown claim for hand calls, see F2.
  - The teardown's `endOwnedBeforeSession` (`pairteardown/teardown.go:334-345`) ends only registered processes. Ties to F2.
- **D10:**
  - `cfg.TestMapUpkeep` is read at `dobbyengine/testmap.go:51`. Survives.
- **D11:**
  - Once the dobby reads the exit record from the prime's `.lym/testcover/`, a missing record means "no exit record" and work left (`dobbyengine/testmap.go:82-84`). The old bolt `testmap-exit.json` is no longer read by the dobby, only by T2, which the record names. Survives.

**Findings**

- **F1 — MEDIUM, design, D5 and D4.**
  - Summary: uncommitted maps block the prime's weft pull.
  - Issue: D5 exempts `_lym/testcover/` only from preflight's clean check. Fabric's weft pull refuses on any tracked uncommitted path through two branches:
    - a weft only behind origin with dirt gets `weftDirtyStop`;
    - a weft that both sides moved has its replay fail with `ErrReplayDirty`, which also ends in `weftDirtyStop`.
  - When this bites: while a pass runs (a full build takes tens of minutes) and after a killed call, the prime's modified tracked maps are that dirt. Any prime sync during that window stops on the weft.
  - The record says nothing about this, and its claim that the leftover is harmless (D5 rationale, D4) does not hold for this path.
  - Fix: state how a weft pull treats uncommitted `_lym/testcover/` paths, such as exempting or auto-committing them before the move, with a bound. Otherwise state the stop and its way forward, `lym fabric commit`.
  - Evidence: `internal/fabricengine/pull.go:406-415`, `:440-445`, `:473-475`.
- **F2 — MEDIUM, design, D9.**
  - Summary: only `--upkeep` calls register, so a hand call in a task is never ended by its pair's teardown.
  - Issue: D9 says "a pass registers … under its worktree's owner" and "a pair's teardown ends … a task's pass". In code, only `request.upkeep` calls `registerUpkeepPass`. `--upkeep` is a hidden flag and refuses package names.
  - Which calls go unregistered: the calls D9 itself describes in a task. These are the operator's or an agent's hand pass without `--upkeep`, and the sweep's named-package rebuild. None of them registers.
  - Consequence: `runproc.End` never reaches them, and the teardown's `RemovePair` runs under a live call. That call's `lockPackage` and `WriteMap` call `MkdirAll` on directories in the removed worktree, then it commits.
  - The D9 acceptance criterion inherits the gap.
  - Fix: state that every call in a task registers under the slug, named rebuilds included, or state what ends an unregistered one and its bound.
  - Evidence: `internal/gatecli/testmap.go:135`, `:238`, `:252-256`, `:681`; `internal/pairteardown/teardown.go:334-336`.
- **F3 — LOW, design, D7.**
  - Summary: the redundancy step reads a current but unrewritten map as stale.
  - Issue: under D7, a map left unwritten keeps its old `Files` and `BuiltAt`. The redundancy report prints `MapAge` from `BuiltAt` and `Drift` from the map's `Files` against the merge base, so a map the last pass judged current shows age and drift it does not have.
  - D7 moves only the readers that compare `commit` with HEAD, and it is silent on these.
  - Fix: state that these readers report the age and drift of the map's last write, or move them to the planned HEAD.
  - Evidence: `internal/gatecli/redundancy.go:315-318`, `:346-364`; `internal/testcover/upkeep.go:77-83`.

**Tool calls**

1. Grep of fabricengine for hard-reset and clean paths, and of `forkrecords.go` for the fork filter: my context lacked the paths that could discard or carry weft state.
2. Grep for the weft pull and commit functions and their dirty handling: my context lacked how a weft pull treats uncommitted tracked paths.
3. Read of `internal/fabricengine/pull.go:370-450`: my context lacked the weft dirty-stop and replay branches that F1 rests on.
