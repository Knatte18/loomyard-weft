```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced every path that writes, resets or reads the state each decision changes. One path breaks D8 and one leaves D8's local-layer migration underspecified; every other path survives.

**Paths traced, per decision**

- **D1, D5 (map writes and reads):** All of these survive.
  - Hand calls of `lym gate testmap` are the only other map writer (`internal/gatecli/testmap.go`, `lockPackage`). D9's lock wait and D9's re-plan after a retake cover them.
  - The redundancy step reads `BuildFailure`. D5's separate failure field leaves that field's meaning as it is.
- **D7 (`test_map_max_age_days`):** Survives. The strict `gateslot` load drops a retired key through `DropRetired` (`internal/configengine/config.go:274`), and `gateslot.MigrateConfig` already exists for the walk.
- **D8, retired `test_map_every` and new `test_map_upkeep`:**
  - Hub base file through `ReconcileHubWideAt`: survives. In its present-file branch, `Migrate` runs before `pruneSparse`, and `applyReinterpretations` runs after the whole call (`internal/hubreconcile/walk.go`).
  - Absent hub base seeded from the prime: survives. `sparseSeed` runs `Migrate` (`internal/configsync/configsync.go:839`).
  - Prime worktree `beam.yaml` through `ReconcileAll` → `reconcileSparseFile`: survives. `Migrate` runs before `pruneSparse` (`configsync.go:674-684`).
  - Local override file through the same function: the `Migrate` order survives, but see finding 2.
  - `lym config reconcile` (`internal/configcli/configcli.go:556`) and clone (`internal/fabriccli/clone.go:112`): survive. Both reach `reconcileSparseFile`, whose `Migrate` call is unconditional.
  - Every `configengine.Load` read before the walk, the prime dobby's `LoadConfig` included: **breaks** (finding 1).
  - `run_settings` and profiles through `PrepareRunSettings`: survives. These are layers of pair worktrees, and only the prime dobby starts passes.
- **D8, dobby state landing count:** `countLandings` writes `LandingsSinceTestMap` (`internal/dobbyengine/tidy.go:145`) and `testMapStep` reads it. Both retire with the count, and the record keeps an old state readable. Survives.
- **D8, `RoleTestMap` registration moving from the dobby to the verb:**
  - Readers: `testMapRunning` (`internal/beamcli/dobbydeps.go:588`), the prime's `lym beam stop` through `EndRole` (`internal/orchcli/cli.go:284`), `EndDobbies` and `dobbyPassInFlight`.
  - A self-registered pid under `HubOwner` reads the same to all of them, and a hand `--upkeep` call is ended by `beam stop` as a dobby-started pass is. Survives.
  - The window before the verb registers itself is covered by D8's stated bound.
- **D8, exit record:** Today it has no production reader. `WriteExitRecord` keeps unknown keys. After an upgrade, a record from the old binary that has no planned HEAD reads as a HEAD that differs, so the dobby starts a pass. Survives.
- **D9 (slots):**
  - Today's hand call takes a slot with the blocking `Acquire` unless it inherits one (`internal/gatecli/testmappackage.go:125-126`), so D9 keeps it as it is.
  - A multi-slot waiter holds the drain lock while it takes slots in order (`internal/gateslot/pool.go`, `AcquireSlots`), and the pass checks `Draining` at each poll and yields. Survives.
- **D11 (anchor move):** Survives.
  - The pass is the dobby's direct child: `proc.Detach` gives it its own process group, and the dobby waits on it (`internal/beamcli/dobbydeps.go:600-614`). `dobbyPassInFlight` matches it by parent pid, so sparing the `RoleTestMap` pid takes it out of the in-flight check.
  - `dobbyOwner(prime)` is `runproc.HubOwner` (`internal/fabriccli/anchordeps.go:92-97`), so `EndSparing` with push and sync spared ends the pass.
  - After the move the dobby restarts, the prior exit record still names the work, and the pass starts again in the new anchor.
- **D12 (exit record fields):** These are added fields under a tolerant reader and writer. Survives.

**Findings**

1. **MEDIUM, design, D8 (`test_map_every` migration).** A hub that set `test_map_every: 0` runs upkeep passes from the binary upgrade until the next walk.
   - `Migrate` runs only inside a reconcile, and the walk runs only from `hubreconcile.Ensure`. Its callers are `lym beam start`, clone, battencli's pair wire and adopt's check.
   - The prime dobby re-execs onto a changed binary between ticks and calls no `Ensure` (`internal/dobbyengine/loop.go:218-261`; `internal/dobbyengine/doc.go:113`).
   - On its next `LoadConfig`, `configengine.Load` drops `test_map_every` silently through `DropRetired` (`internal/configengine/config.go:274`), `test_map_upkeep` fills from the template as true, and the dobby starts passes.
   - D8's rationale says the migration "keeps a hub that set it off, in any layer that set it". That holds only after a walk, and the record does not state this window or bound it.
   - Evidence: `internal/configengine/layers.go:224-254`, `internal/beamcli/cli.go:504-508`.

2. **LOW, design, D8 (`test_map_upkeep`'s category).** D8 writes `test_map_upkeep: false` into beam's local override file but never states the key's `configengine` category.
   - The migrated local file loads only if the key is `CategoryNone`. A local file refuses every other category (PATTERN-config-layers, "Categories"), and `checkLocalDocument` checks local documents.
   - Today `test_map_every` is `CategoryNone` (`internal/dobbyengine/config.go:64`).
   - The record should state that the new key stays `CategoryNone`, so the local-layer migration it promises produces a file the strict beam load accepts.

Tool calls:
1. Grep for `ReadExitRecord` and `LandingsSinceTestMap`, plus `internal/dobbyengine/tidy.go:100-160`: to find every other reader and writer of the exit record and the landing count, which the manifest did not cover.
2. `internal/configreg/runsettings.go:35-110`: to check whether the `run_settings` and profile path that runs a module's `Migrate` reaches beam's retired key.
