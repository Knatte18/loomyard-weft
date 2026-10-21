---
verdict: APPROVED
---

## Alignment at pr-gate: scratch-defeats-test-cache, PR 791 at 03d9700e

The diff serves the board entry's goal: `.scratch/` no longer changes what source-scan tests read and Go keys on.
Fabric wiring now makes the warp root's `.scratch` a directory link into the weft-side `.lyx`, so the module root holds a link whose own `Lstat` stays fixed while files change under its target.

### Against the decision record

- D1: `internal/fabricengine/scratchlink.go` writes the exclude first and stops when that fails.
  It then refuses to link over a tracked `.scratch` through `PathTracked`, and only after that classifies the root.
  The classes are absent, a link compared by `fslink.RawTarget`, a real directory moved by `os.Rename`, and the leave cases.
  The target is built from `RecordsWorktreePath`, `AnchorRel` and `lyxdirs.DotLyxDirName`, never by resolving the warp `.lyx` link.
  `seedScratchExclude` is deleted, and `ExcludeAnchoredDir` stays.
  Wiring runs only through `WireJunctionsWith`, whose production callers are add, checkout and clone.
- D2: the step returns an outcome and never an error, removes nothing, and records `link_created`, `dir_created`, `file_written` and the new `path_renamed`.
  The add, checkout and clone verbs log a leave through `logger.Warn`.
  The mutation oracle and `doc.go`'s Kind list carry `path_renamed`.
- D3: reconcile re-wires through `wireJunctionSet` and runs the scratch step itself on both branches of `repairPairWiring`, without touching Action or Error.
  `applyStaleRemoval` and `Unwire` drop the root scratch name at a root anchor.
  `Remove` unlinks the root link by its own path through `removeLink`, gated by `linkIsFabricOwned`, before the warp worktree removal.
  The health checks are not changed.
- D4: the exclude is `/.scratch` with no trailing slash, a legacy `/.scratch/` line is replaced, and no probe runs.
- D5: `internal/cleanupcli/doc.go` states that the segment works through the link, and the cleanup integration test pins this.
- D6: the `fabric-windows` board entry's body now carries the check on a junction's own modification time and names the scratch link.

The decision record's open risks are covered:

- The Windows junction-time risk is on `fabric-windows`.
- The three Discussion-Review carry-overs are settled in the code:
  - `case-3-moves-tracked-content`: the tracked check runs before any classification.
  - `remove-unlink-blind-on-subpath-anchor`: `unlinkScratchLink` reads the root path itself.
  - `target-creation-and-case-1-refusal-unbounded`: D2's bound holds, and the weft-present guard keeps the step from creating anything without a weft worktree.

The change stays out of `scankit`, the source-scan tests and the agent-facing `.scratch/` path, as the record's Out list requires.

### Overlaps

- `origin-reconcile` (row 14, Webster-Burler-Fix) shares files with this run: `internal/fabricengine/remove.go`, `mutation.go`, `doc.go`, `export_test.go`, `livestate_mutationoracle_test.go`, `internal/cleanupcli/doc.go`, `cleanup_integration_test.go` and `cmd/lyx/uncontainedwrite_test.go`.
  Its decision record and plan do not touch scratch, the scratch exclude, `path_renamed` or `Remove`'s junction sweep.
  This is a text-only overlap: different places in the same files, and both runs add separate lines to shared lists such as the Kind enum and the uncontained-write allowlist.
  Landing order: this run (own row 21) is further along, so it lands first, and `origin-reconcile` takes its changes at Merge-In.
- `quarry-csharp` (row 3) has no changed paths, so it does not overlap.
- `lyx loom peers` lists no landed parent commits.

### Superseded

None.
`test-tiers` depends on this entry and is not made obsolete by it.
