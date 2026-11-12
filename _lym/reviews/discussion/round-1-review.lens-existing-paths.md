```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced every existing code path that writes, reads or resets the state each decision changes.

**Paths traced, per decision**

- **D1:**
  - The map readers and writers all pass `fabricengine.BoltDir` today: `gatecli/run.go:83`, `redundant.go:94`, `redundancy.go:282`, `testmap.go:262`, and the exit-record read in `beamcli/dobbydeps.go:244`. D1 moves them as a group. **Survives.**
  - The prime's weft branch is `main-weft` and the Bolt checkout is `main` (`git worktree list`), so maps on the prime's weft branch stay out of Bolt, as the operator required. **Survives.**
- **D2:** `createWeftWorktreeDroppingRuns` (`forkrecords.go:20-34`) writes everything except the run-records root, so a new pair gets the parent's committed maps. **Survives.**
- **D3:** `integrateWeft` runs only for an anchor change (`anchorchange.go:480`). It replays local weft commits, map commits included, under the new anchor. Nothing merges weft branches, so no map file reaches the parent. **Survives.**
- **D4, locks:** the record says the locks stay under the worktree's `.lym/testcover/`. `ReadMap`/`WriteMap` derive the lock path through `lock.TrackedFileLockPath`. Under `_lym` that path becomes `.lym/locks/_lym/testcover/...`. **The stated location breaks; the locks stay untracked** (finding 2).
- **D4, commit:** `CommitRecordsPaths` takes the weft write lock itself and stages an explicit path list. **Survives.**
- **D4, other records commits:** commits whose pathspec covers `_lym` sweep in maps, which the record already acknowledges. **Survives.**
- **D5:**
  - `preflight.runStateExemptRels` is the only caller of `fabricengine.Clean` that passes exemptions, so D5 changes one place.
  - Webster's dirty check (`websterengine/recordbatch.go:588`) reads the warp root, where `_lym` is excluded. **Survives.**
  - Teardown's `Remove` commits the weft sibling's uncommitted paths before its dirtiness check. **Survives.**
- **D6:** the only map writers are `testMapRun.writeMap` and `writeFailed` (`testmappackage.go:747-776`). Both run after `testMapLocation`, where the owner refusal would go. **Survives.**
- **D7, skipped write:** an unwritten map keeps its old `Format.Environment`. The selector derives the target's go.mod and go.sum hashes from the tree at HEAD (`testselect.go:244-256`), so a stale stamp still misses correctly. **Survives.**
- **D7, failed builds:** `writeFailed` rewrites a kept map with a new `Failure.Commit` at every new HEAD while a build stays broken (`testmappackage.go:751-756`). That is a real content change, so D7's "written as today" covers it. **Survives.**
- **D8:** `state.WriteJSON` always indents (`state.go:94-99`), and `PATTERN-tolerant-records` routes map writes through `internal/state`. D8 therefore needs a compact mode in `internal/state` or a separate writer. This belongs to the D8-evidence question the handler owns; noted only.
- **D9:**
  - `registerUpkeepPass` registers `RoleTestMap` under `runproc.HubOwner`, one registration per hub (`testmap.go:252-254`, `318-352`).
  - The dobby's `TestMapRunning` reads that registration (`dobbyengine/testmap.go:54-60`).
  - **Breaks:** the record says nothing about how a task's pass registers (finding 1).
- **D10:**
  - `beam.yaml` is a per-worktree module with `test_map_upkeep` classed `CategoryNone` (`dobbyengine/config.go:66-85`). The worktree file and the local layer can therefore hold the key, but the record speaks only of a hub's "base file".
  - `MigrateConfig` drops a non-zero `test_map_every` (`config.go:130-135`), so such a layer now reads the new `false` default.
  - No layer on this hub holds either key. **Survives here; the wording is imprecise** (finding 3).
- **D11:** today's map locks sit beside each map in `<bolt>/.lym/testcover/` (`../_bolt/.lym/testcover` listing, `tracked.go:39`), so "its locks" is right for the old directory. After D4 the dobby reads the new exit record, so the old `testmap-exit.json` it keeps is no longer read by the dobby. **Survives.**

**Findings**

1. MEDIUM, design, D9: a task's upkeep pass would register under the hub-wide `runproc.HubOwner` slot, and the record never says whether it does.
   - **Blocks other passes:** while a task runs a pass by hand (up to 44 minutes), the prime's dobby reads the hub as running a pass and starts none. Every other worktree's `--upkeep` call is refused with "the upkeep pass N is running for the hub".
   - **Survives teardown:** pair teardown ends the processes registered under the pair's slug (`runproc.End` per `PATTERN-pair-remote`). It never ends a pass registered under `HubOwner` in a worktree it removes.
   - **Fix:** decide whether a task's pass registers per worktree, or under the task's slug, and state it in D9.
   - Evidence: `internal/gatecli/testmap.go:252-254,318-352`; `internal/dobbyengine/testmap.go:29-30,54-60`; `internal/runproc/record.go:21-24,50-51`.
2. LOW, consistency, D4: the record places the map locks under the worktree's `.lym/testcover/`. `lock.TrackedFileLockPath` puts the lock of a file under `_lym` at `<checkout>/.lym/locks/_lym/testcover/<pkg>/map.json.lock`, following a junction to the weft checkout. They are still untracked, so D4's acceptance criterion holds; only the stated location is wrong.
   - Evidence: `internal/lock/tracked.go:29-44`; `internal/testcover/coveragemap.go:202-213`.
3. LOW, design, D10: the record says "per hub", but `beam.yaml` is a per-worktree module. The prime's worktree file and its local override can also set `test_map_upkeep`, so "base file" understates where the value comes from.
   - A layer still holding a retired non-zero `test_map_every`, which meant on, reads as off after the flip, because `MigrateConfig` drops it and the template now says `false`. The record is silent on this.
   - Evidence: `internal/dobbyengine/config.go:64-85,130-135`; `internal/dobbyengine/template.yaml:4`.

**Tool calls**

1. `integrateweft.go` header, its callers, and the structural-dirs grep: whether any path merges or replays weft content (D3); the manifest lacked it.
2. Hub worktree branches: whether the prime's weft branch is Bolt's `main` (D1, "not in Bolt"); not in context.
3. `fabricengine.Clean` callers and the `testmappackage.go:735-780` write sites: other dirtiness checks (D5) and map writers (D6, D7); only the grep hits were in context.
4. `lock.TrackedFileLockPath`, the users of dirtiness checks, and the old Bolt directory listing: where map locks go before and after D1 (D4, D11).
5. `recordbatch.go` dirtiness, the `testmap.go:244-262` registration condition, and the records-commit scope: webster's dirty check (D5) and which mode registers (D9).
6. The prime and Bolt `beam*.yaml` layers and `configengine` `Dirs`: whether any layer holds the upkeep keys (D10).
