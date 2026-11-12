```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the code paths that also touch the state each decision changes.

**Findings**

- **MEDIUM — D4 (who commits, and the claim that the next pass picks up leftovers).** Other weft commits already sweep `_lym/testcover/` in with them, and the record does not say so.
  - Evidence: `internal/fabriccli/weft_verbs.go:~104-111` builds its commit pathspec from `fabricengine.PathspecNames(ConfigDirs(l))`, which is the whole structural `_lym`. `Remove` commits the weft sibling's uncommitted changes under the record pathspec (`internal/fabricengine/remove.go:~915-935`, and the PATTERN-fabric-boundary text "Remove first commits the sibling worktree's uncommitted changes").
  - In a task pair, a transition or records commit made while a hand-run D7 pass is writing, or after it was killed, carries those maps. The maps are then pushed with that commit, and the pass's own commit never sees them.
  - The record says only "the next pass's commit takes them".
  - The record should say that maps a pass writes may be committed by any `_lym` records commit of the pair. That is harmless because each map file is written whole. It should also say that D4's pass commit is the guaranteed writer, not the only one.
  - D5's owner rule still holds for these sweeps, since records commits already run only on the owner machine.

- **MEDIUM — D4 and D7 (what "a pass" covers).** `lym gate testmap` has two modes: an upkeep pass, and a whole rebuild of named packages. D4 says only "at the end of a pass", so it is unclear whether a named-package rebuild commits and pushes too.
  - Evidence: the testmap Long text, "or rebuilds each named package's map whole", in `internal/gatecli/testmap.go:~96`.
  - If the rebuild does not commit, its maps are left for a later sweep, as in the finding above.
  - The record should state that both modes commit.

- **LOW — D6 (unchanged maps keep their old commit).** This survives the selector's marker check.
  - `testselect.go:224` misses when `Format.Commit != m.Commit`.
  - An unchanged map keeps both values, so the check still passes.
  - Any map the pass does rewrite must stamp `Format.Commit` together with `Commit`, which happens today through the writer.
  - No break, but the record cites none of this. A plan writer who sets only `m.Commit` for an "affected" map would get `MissUnmarked`.

- **LOW — D2 (per-pair size on disk).** `fabricengine`'s uncommitted snapshot lists every uncommitted entry under the record pathspec of the weft worktree (`internal/fabricengine/snapshotuncommitted.go:~78-86`).
  - A pair with a partly written set of maps would snapshot up to about 41 MB.
  - The record's cost note covers the checkout only.

- **LOW — D8 (default off).** `dobbyengine.MigrateConfig` maps the old `test_map_every: 0` to `test_map_upkeep: false`.
  - A hub that relied on the old key being non-zero, or on the template default, now reads off without being told.
  - This is intended, but the record does not say how an operator learns it.
  - There is no reinterpretation and no notice, and the record gives no `## Open risks` line for it.

**Paths traced**

- D1: gate scope, `internal/gatecli/run.go:83` (`request.coverageMaps = testcover.Dir(fabricengine.BoltDir(...))`). It resolves `location` first, so swapping in the worktree's anchor works there, and gate slots read through that location. Survives.
- D1: `lym gate redundant`, the audit, and the dobby's exit-record read (`beamcli/dobbydeps.go`). They read bolt paths today and move with D1 and D4 (Notes). Survives if they move together.
- D2: `createWeftWorktreeDroppingRuns` drops only the `ScopedPathspec(anchor, shedrun.RunsRootRel())` subtree (`forkrecords.go:48-68`). `_lym/testcover` is inherited. Survives.
- D3: weft branches never merge, and teardown archives the tip with a tag. No path brings a task's maps to its parent. Survives.
- D4: the `weft_verbs` records commit and `Remove`'s uncommitted commit both include `_lym/testcover`. Broken as stated; see the finding above.
- D4: `CommitAnchoredPaths` is scoped and positive-only, so it fits D4.
- D5: weft pull and replay on a non-owner machine. That machine never commits maps, so no conflict arises. Survives.
- D6: the selector's marker check (`testselect.go:224`) and reads at `m.Commit` through `CodeFileAt` (`:266`, `:294`, `:318`). The old commit stays readable from the warp store. Survives.
- D6: the dobby's work-left check compares `planned_head` with HEAD, so a map left unchanged does not touch it. Survives.
- D7: the prime-only refusal (`testMapLocation`) is the one gate to widen. Survives.
- D8: `MigrateConfig` and the sparse hub base. The new default applies to every hub that never tuned the key. Survives, but is silent; see the finding above.
- D9: the bolt's exit record is removed. The dobby reads that record today, so the import and the reader's move must land together. Survives, given the Notes line.

**Tool calls**

1. `forkrecords.go` drop logic: to check that D2's fork keeps `_lym/testcover`, since my context had only the agent's summary.
2. `testselect.go` uses of `m.Commit` and `CodeFileAt`: to test D6's claim that the selector diffs from the map's own commit, since the selector body was not in my context.
3. `ScopedPathspec` callers and definition: to find which other weft commits cover `_lym/testcover`.
4. `testselect.go` 215-232, `weft_verbs.go` 90-120, `remove.go` 915-935 and `snapshotuncommitted.go` 60-90: to confirm the marker check and the whole-`_lym` commit and snapshot paths that call 3 found.
5. `gatecli/run.go` 75-90 and the `MapsDir` grep: to check how the gate's coverage scope resolves the maps dir under D1.
