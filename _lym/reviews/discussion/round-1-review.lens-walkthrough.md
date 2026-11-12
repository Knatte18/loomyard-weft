```yaml
lens: walkthrough
outcome: reported
```

Task: walkthrough lens. I ran each decision's mechanism end to end against the code in context.

**Decisions walked**

- **D1:** case: a task worktree's gate reads `<anchor>/_lym/testcover/`. The anchor `_lym` is the pair's weft checkout (`testmap-checked-in/_lym -> testmap-checked-in-weft/_lym`). **Met.**
- **D2:** case: a pair forked from the prime, whose weft branch is `main-weft` (`loomyard-weft`, not the Bolt). `createWeftWorktreeDroppingRuns` drops only the run-records root, so the committed maps come along. **Met.**
- **D3:** case: a task squash-lands. The parent map's commit is an ancestor of the new HEAD, and the selector diffs the landed change. **Met.**
- **D4:** case: an upkeep pass in the prime. It writes the maps, then `CommitRecordsPaths` takes the weft write lock with an explicit path list, then `SpawnPairPush` (= `SpawnStatusPush`) pushes. **Met.**
- **D5:** case: a killed pass leaves maps uncommitted. Adding an entry to `runStateExemptRels` hands it to `fabricengine.Clean`. **Met.**
- **D6:** case: the owner and a second machine on a hub with an `env.machine.yaml` path variable. **Not met.** See F1.
- **D7:** case: a map at H1 is current at H2. Nothing is written, the exit record's `planned_head` is H2, the dobby sees no work, and the next pass diffs H1 to H3. **Met.**
- **D8:** case: the pass writes one map. It goes `WriteMap` → `state.WriteJSON`, which always indents. **Not met as stated.** See F4.
- **D9:** case: an agent runs `lym gate testmap --upkeep` in a task while the prime's dobby is on. **Not met.** See F3.
- **D10:** case: this hub. Its Bolt `beam.yaml` lacks the key, so the template's `false` applies and `testMapStep` returns early. **Met.**
- **D11:** case: the import is killed after writing some of the 211 maps. **Not met.** See F2.

**Findings**

**F1 — MEDIUM, design. D6, and the Constraints fact on the hub env digest.**
- `HubEnvDigest` hashes `envconfig.LoadConfig(fabricengine.ConfigDirs(location)).Entries()`.
- `ConfigDirs` carries both Board and Anchor, so the machine layer stacks.
- env.yaml's `vars` key is `CategoryMachine`, so `env.machine.yaml` values feed the digest. The config-layers background names "an `env.yaml` variable naming a path on its disk" as the kind of value that key holds.
- So the record's "holds no machine name or path, so two machines … read each other's maps as current" is false on any hub with machine-layer env vars. There, every non-owner reads every map as `MissEnvironment` and runs whole packages.
- After `lym adopt` moves the owner, the new owner rebuilds and commits every map, about 44 minutes, and nothing in the record states this.
- Evidence: `internal/envconfig/config.go:65`, `internal/gatecli/testmappackage.go:818`, `internal/gatecli/testmap.go:357-362`, `internal/fabricengine/junctionnames.go:138-143`, `pattern/PATTERN-config-layers.md:26-29`; decision-record.md:63 and :117.

**F2 — MEDIUM, design. D11, a half-finished import.**
- A call killed partway through the import leaves some maps in the prime's `_lym/testcover/`.
- The next call sees "holds a map", so it imports nothing. Its D4 commit commits the partial set. The removal then deletes the rest of the bolt's old maps.
- Those packages read `MissNoMap` and get rebuilt, with no error and no way back. That defeats D11's stated purpose of saving the 44-minute rebuild.
- D11's Bound states no rule for a partial import, such as importing every map missing from the prime's directory or removing only the maps that were imported.
- Evidence: decision-record.md:100-104.

**F3 — MEDIUM, design. D9, a task's upkeep pass registers hub-wide.**
- `lym gate testmap --upkeep` registers as `runproc.RoleTestMap` under `runproc.HubOwner` and refuses while any other process holds that registration.
- The prime dobby's `TestMapRunning` reads that same hub registration.
- So a hand upkeep pass in a task blocks the prime's automatic pass for its whole duration, and a running prime pass refuses every task's pass. The refusal says "the upkeep pass … is running for the hub", which is wrong for a per-worktree pass.
- A task's pass registered under `HubOwner` is also not ended by the pair teardown's `runproc.End(slug)`, so it can outlive its worktree.
- D9 says nothing about which owner a task's pass registers under.
- Evidence: `internal/gatecli/testmap.go:252-254`, `:318-352`; `internal/beamcli/dobbydeps.go:248`; `internal/dobbyengine/testmap.go:54-60`; `pattern/PATTERN-pair-remote.md:58`.

**F4 — LOW, design. D8, compact output.**
- `testcover.WriteMap` writes through `state.WriteJSON`, whose `writeJSONUnlocked` always runs `json.Indent`. `PATTERN-tolerant-records` requires record rewrites to go through `internal/state`.
- So D8 needs a compact mode in the shared `state` writer, a change outside `testcover` that D8 never names.
- "Its keys sorted as today" is also inaccurate: struct fields marshal in declaration order, and only map keys come out sorted.
- Evidence: `internal/state/state.go:73-99`, `internal/testcover/coveragemap.go:34-64`, `:210-213`; decision-record.md:79.

**Tool calls**
1. `ls` of the hub, `git -C _bolt worktree list`, and the junction targets: to find whether the prime's weft worktree is the Bolt, for D2, D4 and D11. The manifest lacked the hub layout.
2. Grep of the env digest and of `testmaptree.go`, plus `testmap.go` 240-265 and 615-640: to trace D6's stamp source and D9's registration and HEAD read, one hop past the manifest.
3. Grep of the `hubEnv` load site, envconfig's `CategoryMachine` and `Entries`, and the hub's `env.yaml`: to check whether the digest can hold machine values.
4. `testmap.go` 355-364 and `fabricengine.ConfigDirs`: to confirm the env load stacks the machine layer, since Dirs carries Board and Anchor.
