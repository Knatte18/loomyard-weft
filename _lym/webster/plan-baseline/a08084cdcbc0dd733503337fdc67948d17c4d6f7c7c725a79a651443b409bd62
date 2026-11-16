# Card 3 — weaver-geometry

**Create:**
- `internal/lymdirs#WeaverDirName`
- `internal/weaverengine#Dir`
- `internal/weaverengine#DirRel`
- `internal/weaverengine#ReportsDir`
- `internal/weaverengine#ScratchDir`
- `internal/weaverengine#PromptsDir`
- `internal/weaverengine#JournalPath`
- `internal/weaverengine#OutcomePath`
- `internal/weaverengine#Geometry`
- `internal/weaverengine#Geometry.runPlan`
- `internal/weaverengine#Geometry.git`
- `internal/websterengine#NewGit`
- `internal/hubgeom#WeaverGeometry`
- `internal/weaverengine_test#TestPaths`
- `internal/hubgeom#TestWeaverGeometry`

**Edit:**
- `internal/lymdirs/doc.go`

**Extends:**
- `internal/weaverengine/doc.go`
- `internal/hubgeom/doc.go`

**Uses:**
- `internal/websterengine/doc.go`
- `internal/websterengine#Git`
- `internal/websterengine#realGit`
- `internal/hubgeom#WebsterGeometry`
- `internal/hubgeom#TestWebsterGeometry`

**Intent:** Give Weaver its run directory, scratch directory and told geometry (D9, D11).

- `lymdirs.WeaverDirName` names the `weaver` segment, so `_lym/weaver/` and `.lym/weaver/` are named through `lymdirs` and nowhere else.
  `lymdirs/doc.go` names `WeaverDirName` beside `UsageDirName` and `RaddleDirName` as the exceptions that sit under a worktree's tracked `LymDirName`.
- `Dir` joins `lymdirs.LymDirName` and `WeaverDirName` onto the anchor; `DirRel` is its forward-slashed `_lym/weaver`, the pathspec the per-batch commit stages.
  `ReportsDir` is `Dir/reports`; `ScratchDir` joins `lymdirs.DotLymDirName` and `WeaverDirName`; `PromptsDir` is `ScratchDir/prompts`.
  `JournalPath` is `<runDir>/journal.md` and `OutcomePath` `<runDir>/outcome.yaml`; the summary path is `summaryparser.Path(runDir)`.
- `Geometry` mirrors `websterengine.Geometry`'s told fields under Weaver's names:
  `AnchorRoot`, `WorktreeRoot`, `PaneCwd`, `RunDir`, `ReportsDir`, `PromptsDir`, `ScratchDir`, `StencilsDir`, `SpecsDir`, `PlanDir`, `VerifyDir`, `GateSlots`, `GateWaitDir`, `GateConfig`, `ParentName`, `DecisionRecordPath`, `Git`, `Index` and `VerifyHold`.
  `runPlan` maps it onto `runplan.Geometry` with `RunDir`, `ReportsDir`, `PromptsDir` and `ScratchDir` naming Weaver's own directories, so Weaver's state lock and state-mutation lease sit under `.lym/weaver/`.
  `ConflictDir` is not a field: Weaver passes `runplan.ConflictDir(AnchorRoot)` (card 1).
- `websterengine.NewGit` returns the real `Git` (`realGit{}`), so Weaver reads its worktree through the same git surface; `Geometry.git` returns `Git` when set and `NewGit()` otherwise.
- `hubgeom.WeaverGeometry` builds the hub-mode geometry from the location as `WebsterGeometry` does, with Weaver's accessors for the run, reports, scratch and prompts directories.
  `hubgeom/doc.go` names it beside `WebsterGeometry`.
- `weaverengine/doc.go` gains its run-directory section: the state file, reports, `journal.md`, `summary.md` and `outcome.yaml` under `_lym/weaver/`; prompts and locks under `.lym/weaver/`.

`TestPaths` is tier0: each accessor's path under a temp anchor, and `DirRel` equal to `_lym/weaver`.
`TestWeaverGeometry` is tier0, as `TestWebsterGeometry` is: every field equals its accessor over the location.

**ImpactSummary:** Adds one lymdirs constant, one exported constructor on websterengine and one hubgeom builder; no existing behaviour changes.
