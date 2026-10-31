# Card 13 — one-discussion-validator

**Create:**
- `plan:internal/loomshed#DiscussionCheck` -> `type DiscussionCheck struct`
- `plan:internal/loomshed#ValidateDiscussion` -> `func ValidateDiscussion(check DiscussionCheck) ([]discussionparser.Finding, error)`

**Edit:**
- `internal/loomshed#NewDiscussionGate` -> `func NewDiscussionGate(check DiscussionCheck) shuttleengine.Gate`
- `internal/loomshed#DarnDeps`
- `internal/loomshed#darnWrite.Call`
- `internal/loomshed#TestNewDiscussionGate`
- `internal/loomshed#TestValidateLoomDiscussion_Citations`
- `internal/loomshed#TestDarnWrite_Call`
- `internal/loomshed#TestGateRows_StopsNameAWayForward`
- `internal/loomshed#TestDiscussionGate_FailureSurfacesItsFindings`
- `internal/loomcli#TestGateParity_DiscussionGate`
- `internal/loomcli#TestSmokeGate_RepromptsThroughARealPaneAndFixesTheArtifact`
- `internal/loomshed/doc.go`
- `internal/shedrecipe#resolvePlainGateClosure`
- `internal/shedrecipe#darnWriteEntry`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#TestResolveGateSpec_ValidOneEntryPerVocabularyName`
- `internal/shedrecipe#TestDarnWriteEntry_ConstructionFailures`
- `internal/shedrecipe#TestResolveGateSpec_Rejections`
- `internal/loomshed#darnWrite.checkSeams`
- `internal/loomshed#newDarnFixture`
- `internal/loomcli#intentionallyNil`
- `internal/battencli#intentionallyNil`
- `internal/shedrecipe#newTestEnv`
- `internal/testkit/envkit#nilLegal`
- `internal/testkit/envkit#FullEnv`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#loomCLI.wireLightweight`
- `internal/loomcli#loomCLI.wireDarn`
- `internal/loomcli#darnDeps`
- `internal/loomcli#loomCLI.validateDiscussionCmd`
- `internal/loomcli#TestValidateDiscussionCmd`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWireLightweight_FillsThePathsWithoutLoadingAnyConfig`
- `contracts/recipes/darn-recipe.yaml`
- `contracts/specs/loom-discussion-spec.md`
- `docs/overview.md`
- `pattern/PATTERN-gate-slot.md`

**Delete:**
- `internal/loomshed#ValidateLoomDiscussion`
- `internal/loomshed#ValidateDarnDiscussion`
- `internal/loomshed#NewDarnDiscussionGate`

**Uses:**
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestFullEnv_PathFieldsAbsoluteUnderOneRoot`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `internal/testkit/envkit#TestSeamMarkers_ReflectiveTestsAreMarked`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/shedrecipe/doc.go`
- `plan:internal/loomrecipe#RecordEngine`

**Intent:** One discussion validator and one gate for every loom record (D5), and Darn-Write's validate-only guard.

`DiscussionCheck` carries `RecordPath`, `SupportLogPath`, `RelevantPath`, `WorktreeRoot`, `Index planindex.Index`, `History func() ([][]byte, error)` and `Citations bool`. `ValidateDiscussion` runs, in order:
1. `discussionparser.Validate`'s section check over the record and support log (card 1's fence rule);
2. the decision-id and acceptance check on a record that exists;
3. when the section check is clean, the history check `discussionparser.CheckAddedDecisionsKept` against `History`, which keeps `Added after Discussion` entries and appended PATTERN bullets on tapestry too;
4. with `Citations` set, the citation check over `Index` and the worktree and the relevant-list check.
Findings come in that order, each stamped with its path; any read, history or quarry error is returned with nil findings. A record whose sections bar `## Goal` are empty passes, since the parser has no empty-section finding. `Citations` follows the record row's engine in the run's recipe, never its name: set exactly when the record row is `DiscussionWrite`.

- `NewDiscussionGate(check)` maps `ValidateDiscussion` onto the gate contract as today's loom gate does (findings re-prompt, an error returns as is) and commits nothing. `ValidateLoomDiscussion`, `ValidateDarnDiscussion` and `NewDarnDiscussionGate` go.
- `resolvePlainGateClosure`'s `"discussion"` requires the record and support-log paths absolute and `Env.DiscussionHistory` non-nil, and, when `Env.RecordEngine` is `DiscussionWrite`, the relevant path, worktree root and plan index; it no longer branches on `DiscussionHistory`'s presence.
- `Env` gains `RecordEngine string`, the engine of the run's record row, and `DiscussionHistory`'s doc says every loom wiring sets it. `envkit`'s `nilLegal` drops `DiscussionHistory`; `FullEnv` and `newTestEnv` fill both. battencli's `intentionallyNil` lists `DiscussionHistory`, since batten wires no loom record.
- `wire` and `wireLightweight` set `DiscussionHistory` and `RecordEngine` (from `loomrecipe.RecordEngine(c.recipe)`, empty recipe read as tapestry) and `PatternRecordPath`'s darn branch stays for card 18; `wireDarn` stops setting them.
- `lym loom validate-discussion` runs `ValidateDiscussion` over the same fields (Gate Self-Check Parity) and commits nothing on either recipe; its help text drops the darn commit. A Discussion-written record with an unclosed fence now reports every later section missing, and the gate re-prompts its writer to close the fence.

Darn-Write's guard:
- `darn-recipe.yaml`'s Darn-Write row drops its `discussion` gate; `darnWriteEntry` no longer resolves one.
- `DarnDeps` gains `CheckRecord func() ([]discussionparser.Finding, error)`, which `darnDeps` wires to `ValidateDiscussion` over the run's record with `Citations` false. It is a required seam: `darnWrite.checkSeams` names a nil one, and `darnWriteEntry` refuses it through `requireSeam`, so a hand-built receiver must fill it; `newDarnFixture` fills it with a clean check. `darnWrite.Call` runs it once the session returns, after the landing commit: findings halt the run Stuck, with no re-prompt (Record-Write never overwrites a file that exists, so a re-prompt could only push the writer to edit the record). The reason names the findings and the way forward `restore each failing file, in the _lym weft repository, to its latest version "lym loom validate-discussion" accepts (the committed copy when the damage is uncommitted, restored with "git -C _lym checkout HEAD -- discussion/<file>"), commit it with "lym fabric commit", which the orch runs when a session lym refuses it from reports status: FAILED, then run "lym loom resume"`, since restoring keeps every `Added after Discussion` entry the record held. It commits nothing; an error is returned.

Tests:
- `TestNewDiscussionGate`: one validator serves both forms. A Record-Write record passes; a record whose body holds `## Open risks` and whose own `## Open risks` is removed is refused; a record that drops an `Added after Discussion` entry its history held is refused with `Citations` set and unset; a ```` ```js ```` line inside an open fence before a `## Decisions` line, and a fence indented three spaces, both validate with that line fenced.
- `TestValidateLoomDiscussion_Citations` calls `ValidateDiscussion` with `Citations` set and keeps its citing rows (missing path, malformed relevant list).
- `TestDarnWrite_Call`: a guard finding after the session halts Stuck naming the findings, `lym fabric commit` and `lym loom resume`, with one session spawn; a clean guard is Done.
- `TestValidateDiscussionCmd`: its darn row reports the same findings as the gate and commits nothing.
- `TestGateRows_StopsNameAWayForward` covers the guard's reason; `TestResolveGateSpec_ValidOneEntryPerVocabularyName` builds `discussion` both ways; `TestResolveGateSpec_Rejections` gains a row with a nil `Env.DiscussionHistory` refused by field name; `TestDarnWriteEntry_ConstructionFailures` drops its discussion-gate case and gains a nil `Darn.CheckRecord` row; `intentionallyNil` in loomcli and battencli lists `Darn.CheckRecord`, which only a darn seed fills, and battencli's also lists `DiscussionHistory`, so `TestWire_EverySeamFilled` holds both ways; `TestWire_DarnRecipe` and `TestWireLightweight_FillsThePathsWithoutLoadingAnyConfig` assert `DiscussionHistory` and `RecordEngine` on both recipes.
The discussion spec says the history check runs on every loom record, the citation and relevant-list checks follow the record row's engine, and `validate-discussion` commits nothing; `docs/overview.md`'s validate-discussion lines follow. `pattern/PATTERN-gate-slot.md`'s Pairs section names `loomshed.ValidateDiscussion` as the function Discussion-Write's and Discussion-Burler-Fix's gates and `validate-discussion` all call, and drops the Darn-Write `discussion` gate pair, since Darn-Write's check is now a post-session guard.

**ImpactSummary:** Two validators and two gates become one; the gate resolver, both wirings, the verb and Darn-Write's producer switch to it, and darn's writer gate becomes a post-session guard.
