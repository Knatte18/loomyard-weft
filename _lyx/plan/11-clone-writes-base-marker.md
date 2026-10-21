# Card 11 — clone-writes-base-marker

**Create:**
- `plan:internal/fabriccli_test#TestRunCLI_CloneOfAnUnmigratedWeftLeavesTheMarkerToTheWalk` -> `func TestRunCLI_CloneOfAnUnmigratedWeftLeavesTheMarkerToTheWalk(t *testing.T)`

**Edit:**
- `internal/fabriccli#CloneAndWire`
- `internal/fabriccli#runReconcile`
- `internal/fabriccli_test#TestRunCLI_CloneEndToEnd`
- `internal/fabriccli_test#TestRunCLI_HubMutationScenario`
- `internal/hubforge#TestNewHub`

**Uses:**
- `plan:internal/configsync#WriteBaseSparseMarker`
- `plan:internal/configengine#BaseSparseMarker`
- `plan:internal/configengine#BaseSparseMarkerRel`
- `plan:internal/configengine#SparseBaseHeader`
- `internal/fabriccli_test#makeCLICloneWarpBare`
- `internal/fabriccli_test#makeCLICloneWeftBare`
- `internal/fabriccli_test#gitOutputCLI`

**Intent:**
`lyx fabric clone` marks a board it seeded from nothing, in its own board commit (D3).
`CloneAndWire` records, before its `ReconcileHubWideAt` pass, whether any registered module's base file existed at the board dir; when none did, after the pass it calls `configsync.WriteBaseSparseMarker` on the board dir and appends a `KindFileWritten` entry for `configengine.BaseSparseMarker` to its recorder (`PATTERN-mutation-record`), so the whole-repo board commit that follows, "fabric clone: record anchor + repo-wide config", carries the marker beside the sparse hub files.
Every file that pass seeds is already sparse (card 6), and the clone's pass runs without `SeedDefaults`, so the per-worktree repo defaults are still absent when the marker is written; the walk the clone then runs seeds them sparse after the marker, and the walk's own precondition that every base exists does not apply to the clone's marker.
A clone over a weft whose board already holds a base file writes no marker, so its walk, which runs right after with no stamp, strips that unmigrated board once and writes the marker in the walk's board commit.
`CloneAndWire`'s comment on the walk it runs, which says the walk "seeds every absent per-worktree repo default from the prime or the template", says the seed is sparse: the header-only base from the template, the prime's non-template units from the prime.
The `ReconcileHubWideAt` comment in `runReconcile` that says it "only adds absent keys" says instead that it seeds absent hub files sparse, prunes keys the template lacks and never rewrites a recorded pathspec, and strips no present base file.

`TestRunCLI_CloneEndToEnd` asserts that the board tracks `configengine.BaseSparseMarkerRel()`, that the commit adding it is the clone's "fabric clone: record anchor + repo-wide config" commit, that the envelope's mutation record holds a `file_written` entry for `configengine.BaseSparseMarker`, and that the board's `fabric.yaml` holds no template unit.
`TestRunCLI_CloneOfAnUnmigratedWeftLeavesTheMarkerToTheWalk` builds a weft bare whose `main` holds a full `_lyx/config/fabric.yaml` at the template's values on the board side, no marker, and a tracked prime `_lyx/config/loom.yaml` at the pre-change values (the current template with `discussion_lenses: ""`) and no worktree `sparse` marker; it clones it, and asserts the board marker is tracked and was added by the walk's "lyx: reconcile hub-wide config for build" commit, that `fabric.yaml` was stripped to the header, that the prime's `loom.yaml` no longer holds `discussion_lenses`, and that `loomengine.LoadConfig` over the prime resolves `DiscussionLenses` `twill-discussion`.
`TestRunCLI_HubMutationScenario`'s reconcile step gains an assertion over a board whose `board.yaml` it rewrites to the template whole and whose marker it removes before `lyx fabric reconcile`: the file keeps its units and no marker is written, since only the walk strips.
`TestNewHub`'s `ConfigMaterializedWithoutSeeding` step also asserts the board tracks the marker.

**ImpactSummary:** `CloneAndWire` writes and records one more board file on a fresh board; the clone's board commit and the hubforge fixtures built through it carry it, and the walk covers every other board.
