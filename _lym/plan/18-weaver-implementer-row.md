# Card 18 — weaver-implementer-row

**Create:**
- `internal/shedrecipe#weaverEntry`
- `internal/shedrecipe#TestWeaverEntry`
- `internal/testkit/shedfake#WeaverSeams`

**Edit:**
- `internal/shedrecipe#implementers`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#websterEntry`
- `internal/shedrecipe#TestImplementerEntry_RefusesAnUnusableSetting`
- `internal/shedrecipe#TestImplementerEntry_ForwardsTheRestOfTheRowConfig`
- `internal/shedrecipe#newTestEnv`
- `internal/loomcli#loomCLI.wire`
- `internal/testkit/envkit#FullEnv`
- `internal/testkit/envkit#nilLegal`
- `internal/battencli#intentionallyNil`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/shedrecipe/doc.go`

**Uses:**
- `internal/shedrecipe#resolveGateSpec`
- `internal/weaverengine#ResolveLadder`
- `internal/testkit/envkit#TestFullEnv_PathFieldsAbsoluteUnderOneRoot`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `internal/testkit/envkit#TestSeamMarkers_ReflectiveTestsAreMarked`

**Intent:** `tapestry.implementer: weaver` builds the implementer row with Weaver (D1, D5).

- `implementers` gains `"weaver": weaverEntry`; the default `tapestry.implementer` stays `webster`.
- `weaverEntry` resolves the row's `gates` key through `resolveGateSpec` onto `RunDeps.Gate`, rejects any other key, requires `ConfigDirs.Anchor`, `WeaverRun` and the `WeaverDeps` seams `Runner`, `Fixer`, `Reed`, `CommitRecords` and `WeftHead`, loads `weaver.yaml` and resolves its ladder with `weaverengine.ResolveLadder` over `Env.Models`.
  An empty ladder or an unresolvable spec refuses when the row builds, naming `lym config weaver`.
  It returns `loomshed.NewWeaverProducer` with the seam deps from `Env.ImplementerSeam`.
- `Env` gains `WeaverRun shedadapters.WeaverRunner`, `WeaverDeps weaverengine.RunDeps` and `ImplementerSeam weaverengine.ActiveDeps`; `websterEntry` passes `ImplementerSeam` to `NewWebsterProducer` (card 17).
- `loomCLI.wire` fills them: `WeaverRun` with `weaverengine.Run` explicitly, as `WebsterRun` is set; `WeaverDeps` over `hubgeom.WeaverGeometry` with the code index and pause hold set as Webster's are, the shuttle runner as `Runner`, `gatefixer.NewStrandFixer` with the runproc registrar as `Fixer`, the reed engine, the language skills, the friction directory, `ParentBranch` as Webster's reads the recorded parent branch, `CommitRecords` committing `weaverengine.DirRel()` and the plan directory through `fabricengine.CommitAnchoredPaths`, and `WeftHead` from `fabricengine.CheckoutStates`' records head;
  `ImplementerSeam` with the anchor, the `tapestry.implementer` setting lookup, `fabricengine.AnchoredFileFirstParentHistory` over `StateRel()` mapped onto `weaverengine.StateVersion`, the task branch's head and the reed engine's live strand names.
  `Deliver` stays nil here; card 21 fills it.
- Test fixtures fill the new fields: `newTestEnv`, `envkit.FullEnv` (with `shedfake.WeaverSeams`), `nilLegal` for Weaver's optional seams (`Verify`, `Clock`, `Deliver`, `ParentBranch`, `Geom.Git`, `Geom.GateSlots`, `Geom.GateConfig`), and batten's `intentionallyNil` names the new fields loom-only.
  Every seam-marked test that reaches the `Env` set gains `//lym:seam internal/weaverengine#RunDeps`, `internal/weaverengine#Geometry` and `internal/weaverengine#ActiveDeps`: the four loomcli wiring tests, batten's `TestWire_EverySeamFilled` and envkit's `TestFullEnv_FillsEverySeam`.
- `shedrecipe/doc.go`'s Implementer section names the `weaver` value.

`TestWeaverEntry` is tier0: a full env builds the Weaver producer; each required seam missing refuses naming it; an unknown key refuses; an empty ladder and an unknown alias refuse naming `lym config weaver`; a `gates` entry reaches `RunDeps.Gate`.
`TestImplementerEntry_RefusesAnUnusableSetting`'s rows now list `webster, weaver` as the accepted values, and `TestImplementerEntry_ForwardsTheRestOfTheRowConfig` gains a `weaver` row building with `gates`.

**ImpactSummary:** Adds a second value to the closed implementer table and three Env fields, filled by loom's wiring and every Env fixture and seam-marked test; Webster's row is unchanged.
