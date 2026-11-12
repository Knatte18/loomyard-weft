```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked every bound, guarantee and claim about existing code in the decision record. Three hold up only partly and three break.

**Claims attacked**

- D1, no new branch, worktree or junction: held.
- D2, the fork keeps `_lym/testcover/` and drops only run records: held (`forkrecords.go:1-34`).
- D3, two tasks never conflict on a map file: held. `integrateweft.go` only replays a branch's own commits onto its own upstream and merges no branch into another.
- D4, the verb's own commit covers only `_lym/testcover/`, under the records write lock, with a positive pathspec: held (`commitweftpaths.go:21-41`).
- D4, a killed call's maps are committed by the next call: held as stated.
- D4, another `_lym` records commit sweeping in maps is harmless because each file is written whole: held (`state.go:99`, atomic write).
- D4, the map locks stay under the worktree's `.lym/testcover/`: **broken** (finding 5).
- D4, the task pair's push goes through `PATTERN-pair-remote`: held for the push. The owner-machine clause on a run's own push is the handler's to judge.
- D5, the exemption covers that subtree alone: held.
- D6, only the owner machine writes maps: held. `gatecli/testmappackage.go:753,772` is the only `WriteMap` caller.
- D6, two machines with the same toolchain and hub config read each other's maps as current: **broken** (findings 1 and 2).
- D6, the no-owner bound: held as stated.
- D7, the commit a map stays at stays reachable through warp history: **partly holds** (finding 4).
- D7, a map left at an older commit is still read as current: **broken** for the audit (finding 3).
- D7, the list of conditions that skip a write is complete: **broken** (finding 6).
- D8, the reader accepts the indented form and the version stays 2: held. `ReadJSON` ignores whitespace and `mapMiss` checks only `Format.Version`.
- D8, "its keys sorted as today": **partly holds** (finding 7).
- D9, a task's pass commits only to the task's weft: held for the commit. The pass's registration is a gap (finding 8).
- D10, a hub whose base file holds the key keeps its value: holds, but it is incomplete (finding 9).
- D11, a second call imports nothing: **breaks** after a killed import (finding 10).
- D11, the import runs in the prime only: held.

**Findings**

1. **MEDIUM**, design. D6 and the Constraints Fact on the hub env digest.
   - The record says the digest holds no machine name, so machines with the same toolchain and hub config share maps.
   - `stampEnvironment` hashes `hubEnv.Entries()` (`gatecli/testmappackage.go:818`) from `envconfig.LoadConfig(dirs)` (`gatecli/testmap.go:362`).
   - env.yaml's `vars` key is `CategoryMachine` (`envconfig/config.go:65`), so the per-machine `env.machine.yaml` layer is stacked into that digest.
   - `PATTERN-config-layers` gives "an `env.yaml` variable naming a path on its disk" as a machine-layer example.
   - Counterexample: two machines that each set one path variable in `env.machine.yaml` read every map of the other as `MissEnvironment`.
   - So the sharing premise and the Fact both leave out the machine layer.

2. **MEDIUM**, design. D6 sharing across machines in the prime.
   - A map is usable only if its commit is in the reader's object store. `commitMiss` gives `MissCommitUnreadable` otherwise (`testcover/testselect.go:260-273`), and `PlanUpkeep` turns that into `UpkeepMiss` and a whole rebuild (`testcover/upkeep.go:22,74`).
   - The prime's push is records-only, and its code branch is the operator's (`fabricengine/spawn.go:156-170`, `PATTERN-pair-remote`).
   - Counterexample: the owner commits maps at a prime HEAD the operator has not pushed. A non-owner pulls the weft, lacks that commit, and misses on every package. D6 states neither this condition nor that outcome.

3. **MEDIUM**, design. D7 against the audit.
   - The audit skips any map whose commit is not HEAD: `m.Commit != r.head` drops it from the candidates (`gatecli/testmapaudit.go:38`).
   - Under D7, a current map keeps its older `commit`, so every unchanged package drops out of the audit.
   - D7's claim "the next pass still plans it as current" holds for the planner, but this reader misreads the new meaning of `commit`, and the record says nothing about it.

4. **LOW**, design. D7's Bound.
   - The record says warp history keeps a map's commit reachable but cites no evidence, and does not say what happens when it is not.
   - Evidence exists: the `fabricengine` package doc says a code branch is never rebased (`fabricengine/doc.go:201`). It also says an upstream rewrite is detected (`doc.go:18`) and that its objects survive a fetch (`doc.go:43`).
   - The failure mode is bounded: after an operator force-push and a teardown's `git gc --auto`, the selector gives `MissCommitUnreadable`, the package is rebuilt whole, and the map is rewritten.
   - The record should say so instead of resting on an unbacked claim.

5. **LOW**, consistency. D4's "the locks ... stay never-tracked, under the worktree's `.lym/testcover/`".
   - For a map under `_lym`, `lock.TrackedFileLockPath` puts the lock at `<checkout>/.lym/locks/_lym/testcover/<pkg>/map.json.lock` (`lock/tracked.go:29-36`), not under `.lym/testcover/`.
   - Only the build locks and the exit record sit there. The locks stay untracked either way, so only the stated path is wrong.

6. **LOW**, design. D7's skip condition.
   - D7 skips the write when the plan has "no dropped entry, no queued test and no cleared failure".
   - `PlanUpkeep` also adds an entry without queueing it: a newly declared test the spawn scan marks out of process or unresolved (`testcover/upkeep.go:114-117`).
   - Counterexample: an implementation that follows D7's list literally never writes that entry and replans the test on every pass.
   - D7 should say "no change to the entries".

7. **NIT**, consistency. D8's "its keys sorted as today".
   - `json.Marshal` emits struct fields in declaration order and sorts only map keys (`state.go:78`), so "sorted" is not accurate today.
   - D8 also does not say that `state.WriteJSON` always indents (`state.go:94-99`), so compact output needs a change to `internal/state`. `PATTERN-tolerant-records` says records are rewritten through `internal/state`.

8. **MEDIUM**, design. D9's task pass and the hub-wide registration.
   - An upkeep pass registers as `runproc.RoleTestMap` under `runproc.HubOwner` and refuses while another holds it (`gatecli/testmap.go:318-352`). The dobby reads any registered pass as live (`dobbyengine/testmap.go:29-30,54-60`).
   - Counterexample: a 44-minute upkeep pass run by hand in a task blocks the prime's upkeep, and the reverse.
   - Because it is registered under `HubOwner` and not the task's slug, a pair teardown's `runproc.End(slug)` does not end it. The pass keeps writing into a worktree being removed.
   - D9 does not state either bound.

9. **LOW**, consistency. D10's layer claim.
   - beam.yaml is a per-worktree module, and `test_map_upkeep` is `CategoryNone`, open to every layer (`dobbyengine/config.go:66-77`).
   - The prime's worktree file or its local override can also hold the key and keep its value. "A hub whose base file holds the key" names only one of those layers. The effect is unchanged, so this is wording only.

10. **MEDIUM**, design. D11's import bound after a killed import.
    - The import runs only while the prime's directory holds no map, and the old maps are removed once the prime's maps are committed.
    - Counterexample: a call killed halfway through the import leaves some maps in `_lym/testcover/`. The next call imports nothing, commits that partial set, and then deletes the rest from `<bolt>/.lym/testcover/`.
    - Those packages then read `MissNoMap` and need the rebuild D11 exists to avoid. D11 does not state this bound.

**Tool calls**

- Bash: read `lock.TrackedFileLockPath`, the env-digest site, how the audit and redundancy readers use a map's commit, the head of `integrateweft.go`, and `SpawnStatusPush`. My context had none of them, and each is one hop past a claim in the record.
- Bash: read how `envconfig.LoadConfig` and `Entries` load the env digest's layers, and the audit's candidate filter. They decide whether the digest picks up the machine layer and how the audit reads `commit`.
