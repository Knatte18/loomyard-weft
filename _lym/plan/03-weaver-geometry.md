# Card 3 — weaver-geometry

**Create:**
- `plan:internal/lymdirs#WeaverDirName` -> `const WeaverDirName = "weaver"`
- `plan:internal/weaverengine#Dir` -> `func Dir(anchorRoot string) string`
- `plan:internal/weaverengine#DirRel` -> `func DirRel() string`
- `plan:internal/weaverengine#ReportsDir` -> `func ReportsDir(anchorRoot string) string`
- `plan:internal/weaverengine#ScratchDir` -> `func ScratchDir(anchorRoot string) string`
- `plan:internal/weaverengine#PromptsDir` -> `func PromptsDir(anchorRoot string) string`
- `plan:internal/weaverengine#JournalPath` -> `func JournalPath(runDir string) string`
- `plan:internal/weaverengine#OutcomePath` -> `func OutcomePath(runDir string) string`
- `plan:internal/weaverengine#Geometry` -> `type Geometry struct`
- `plan:internal/weaverengine#Geometry.runPlan` -> `func (g Geometry) runPlan() runplan.Geometry`
- `plan:internal/weaverengine#Geometry.git` -> `func (g Geometry) git() websterengine.Git`
- `plan:internal/websterengine#NewGit` -> `func NewGit() Git`
- `plan:internal/hubgeom#WeaverGeometry` -> `func WeaverGeometry(l *lymcwd.Location) weaverengine.Geometry`
- `plan:internal/weaverengine#TestPaths` -> `func TestPaths(t *testing.T)`
- `plan:internal/hubgeom#TestWeaverGeometry` -> `func TestWeaverGeometry(t *testing.T)`

**Extends:**
- `internal/weaverengine/doc.go`
- `internal/hubgeom/doc.go`

**Uses:**
- `internal/lymdirs/doc.go`
- `internal/websterengine/doc.go`
- `internal/websterengine#Git`
- `internal/websterengine#realGit`
- `internal/hubgeom#WebsterGeometry`
- `internal/hubgeom#TestWebsterGeometry`

**Intent:** Give Weaver its run directory, scratch directory and told geometry (D9, D11).

- `lymdirs.WeaverDirName` names the `weaver` segment, so `_lym/weaver/` and `.lym/weaver/` are named through `lymdirs` and nowhere else.
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
