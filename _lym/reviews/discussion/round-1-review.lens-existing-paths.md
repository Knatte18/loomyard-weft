```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the code that already writes or reads each piece of state the record changes, and checked whether each decision survives it.

**Paths traced, by decision**

- **D1, re-stamp moves `Commit` and `Files`:** `gatecli/redundancy.go:294-315` `mappedPackages` reads `Commit`, `BuiltAt` and `Files` (`mapDrift`). Survives, but `MapAge` now shows the age of the re-stamp, not of the measurement. That is cosmetic.
- **D1, plan skips a map whose build lock is held:** `gatecli/testmap.go:466` `lockPackage` takes the lock with `TryAcquire`. Survives.
- **D1, interim rule:** `testmap.go:456` `stale` stays until the build lands. Survives.
- **D4, spawn verdicts:** `testcover/spawnscan.go:175` `ScanSpawns` gives every tier0 test an empty verdict. Survives.
- **D4/D5, redundancy step:** `redundancy.go:305` refuses a map with `BuildFailure` set; D5 keeps its failure in a field of its own. Survives.
- **D5, the stale check that reads `BuildFailure` at another HEAD:** `testmap.go:456`, replaced by the D5 field. Survives.
- **D7, `test_map_max_age_days` retired in gateslot:** strict loader, and `RetiredKeys` already exists (`gateslot/config.go:164`). The key is `CategoryCrossPair`. Survives.
- **D8, `test_map_batch` retired:** `dobbyengine/config.go:102`, the floor check, goes with it. Survives.
- **D8, `LandingsSinceTestMap`:** written by `dobbyengine/tidy.go:145` and read by `testmap.go:31`. Retired; the Constraint keeps it readable. Survives.
- **D8, `test_map_every: 0` carried by the reinterpretation through the hub walk:** `hubreconcile/walk.go:232` runs `configsync.ReconcileHubWideAt`, whose `pruneSparse` (`configsync.go:1157`, `:727`) drops retired keys. Only then does `walk.go:254` run `applyReinterpretations`. **Breaks.**
- **D8, `test_map_every` in the prime worktree's `beam.yaml` or its `beam.local.yaml`:** `configengine/layers.go:135-146` stacks both for the per-worktree `beam` module. `hubreconcile/reinterpret.go` rewrites only `ConfigFile(boardDir, …)`. **Breaks.**
- **D8, exit record written only by a pass the dobby starts:** today `testmap.go:482` `close` writes it on every call. Production code has no reader (no `ReadExitRecord` caller). Nothing in the record says how the verb tells a dobby start from a hand call. **Gap.**
- **D8/D9, bare hand call `lym gate testmap`, naming no packages:** listed in the verb's examples (`testmap.go:83`). Neither D8 nor D9 says whether it plans and yields like a pass or takes a slot blocking like a hand call. **Gap.**
- **D9, slot inherited through `InheritEnv`:** `testmappackage.go:125`, `gateslot/pool.go:381` `Inherited`. Survives.
- **D9, yield reads `Holders` and `Draining`:** `pool.go:338` and `:374`. Survives.
- **D9, hand call's slot hold against its new wait for a package's build lock:** `testmappackage.go:113-139` `holdSlot` keeps the slot to the end of the call. D9 makes the call wait for a build lock that a yielded pass may hold while it waits for two free slots. **Breaks** (see finding 4).
- **D10, ordering from `RunRecord`:** `gateslot/runlog.go:113` `PackageLaunch.ElapsedSeconds` exists, filled only for go-json tiers and pruned by `runLogMaxAge`. "Every other queued package" covers the rest. Survives.
- **D11, ending a pass on an anchor move:** the wait is in `fabriccli/anchor.go:448-460` (`passesInFlight` / `dobbyPassInFlight`), not in the dobby. After the wait, `anchordeps.go:255-268` `EndDobbies` already ends every `HubOwner` process except push and sync through `runproc.EndSparing`. **Misplaced** (see finding 3).
- **D12, `Unbuilt` replaced:** no production reader of `ExitRecord`, and the verb's `unbuilt` output key has no reader in dobbyengine, beamcli or orchcli. Survives.
- **D6, the environment tests run in:** `testmap.go:265` `gateEnv(environ, hubEnv)` adds `env.yaml` variables, which D6's stamp does not digest. **Gap.**

**Findings**

1. **BLOCKING · design · D8 and the PATTERN-config-layers Constraint.** The reinterpretation of `test_map_every: 0` never sees the key, because the hub walk prunes retired keys before it applies reinterpretations.
   - The Constraint retires `test_map_every` as a retired key of its module.
   - `ReconcileHubWideAt` runs `pruneSparse` over every module's present hub base file. Without `pruneUnknown`, `pruneSparse` still removes the units a retired key covers (`configsync.go:719-739`, called at `:1157`).
   - That happens at `walk.go:232`, and `applyReinterpretations` runs after it at `walk.go:254`.
   - So the rewrite to `test_map_upkeep: false` reads a file that no longer holds `test_map_every`. A hub that turned upkeep off gets it switched back on, silently.
   - `configreg.Module.Migrate` runs before the prune (`configsync.go:1140`) and is documented as rewriting retired keys (`configreg.go:77-78`). It is the existing path that carries a retired key's value.
   - Evidence: `internal/hubreconcile/walk.go:232`, `:254`; `internal/configsync/configsync.go:722-739`, `:1157`.

2. **MEDIUM · design · D8.** The reinterpretation reaches only the hub base file, yet `beam` is a per-worktree module whose prime worktree file and local file can also hold `test_map_every: 0`.
   - `LayerFile` stacks the worktree file `_lym/config/beam.yaml` and the local file for a module that is not hub-wide (`layers.go:135-146`).
   - `test_map_every` is `CategoryNone`, so the local layer admits it (`dobbyengine/config.go:64`).
   - `applyReinterpretations` rewrites only `configengine.ConfigFile(boardDir, module.Name)` (`reinterpret.go:51`).
   - The worktree and local reconciles drop the retired key (`configsync.go:684`).
   - So an off switch set in the prime's own file is lost, and D8 states no bound for this.
   - No such file holds the key on this hub today. I checked the worktree `beam.yaml` files and found no `beam.local.yaml`.

3. **MEDIUM · design · D11.** D11 puts the end of a pass in the dobby, but the anchor move's wait and its process end live in `lym fabric anchor`.
   - D11 says "the dobby ends it through `runproc.EndRole` … and proceeds once it has ended". The waiter is the anchor verb's `passesInFlight` loop (`fabriccli/anchor.go:448-460`), which counts the refresh as the dobby's live child (`anchordeps.go:455-490`).
   - The same verb already ends `HubOwner` processes other than push and sync once the wait is over (`anchordeps.go:255-268`, called at `anchor.go:468`).
   - The fix is that `dobbyPassInFlight` stops counting a `RoleTestMap` child as a pass in flight, so the existing `EndDobbies` ends it.
   - As written, an implementer adds an end step to dobbyengine, and the move still waits in fabriccli.

4. **MEDIUM · design · D9.** A hand call can wait forever on a build lock that a yielded pass holds.
   - `holdSlot` keeps the slot from the first build to the end of the call (`testmappackage.go:113-139`, `testmap.go:479-488`).
   - Under D9, a hand call naming A and B builds A while holding a slot, then waits for B's build lock.
   - A pass that yielded mid-job on B waits to retake a slot "under the same rule", which needs two free slots.
   - On a two-slot hub, or a three-slot hub with one gate running, the hand call's own slot keeps fewer than two free. Each process waits on the other until HEAD moves or a signal arrives.
   - The record says neither whether a yielded pass keeps the package's build lock, nor what bounds the hand call's lock wait.
   - The redundancy sweep (D13) is such a hand call.

5. **MEDIUM · design · D8/D9.** Two existing behaviours of the verb are left unspecified.
   - `lym gate testmap` with no package named is a documented hand form (`testmap.go:74-85`). D9 governs only "a hand call … naming packages", and D8 only a pass the dobby starts. The bare call's slot rule, whether it yields, whether it plans, and whether it writes the exit record are unstated.
   - "Only a pass the dobby starts reads and writes the exit record" needs the verb to tell a dobby start from a hand call. Today the dobby passes only `--max` (`beamcli/dobbydeps.go:604`), which D8 retires. So a new flag or marker is implied. D8 does not name it or state what it lets a hand caller do, such as overwriting the exit record by passing it.

6. **LOW · design · D6.** The stamp leaves out the hub `env.yaml` variables that testmap adds to every test's environment.
   - `prepare` builds the children's environment as `gateEnv(r.request.environ, hubEnv)` from `envconfig` (`testmap.go:237-265`).
   - D6 digests only "the mapped tiers' command and environment as configured".
   - A change to `env.yaml` changes what tests run under, with no miss and no rebuild.

Tool calls:

1. Grep for readers of `ReadExitRecord`, `ReadMap`, `BuiltAt`, `LandingsSinceTestMap`, the age key and the build lock: the manifest had definitions, not every reader.
2. Grep dobbyengine for the anchor move's pass-in-flight wait: the manifest held only the doc's Bound line.
3. Grep `HubOwner` users and exit-record readers across modules: to find the anchor path and any `Unbuilt` reader.
4. Grep `internal/fabriccli/anchor*.go` for `runproc` and in-flight use: the anchor verb was beyond the manifest.
5. Read `internal/fabriccli/anchor.go:440-480` and `anchordeps.go:245-275`, `:455-490`: the wait loop, `EndDobbies` and `dobbyPassInFlight`, one hop past D11's citation.
