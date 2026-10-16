# Card 11 — plan-gate-context-checks

**Create:**
- `plan:internal/loomshed#PlanContext` -> `type PlanContext struct`
- `plan:internal/loomshed#planContextFindings` -> `func planContextFindings(plan *planparser.Plan, cards []planparser.Card, pc PlanContext) ([]planindex.Finding, error)`
- `plan:internal/loomshed#renameToolMissingCheck` -> `const renameToolMissingCheck = "rename-tool-missing"`
- `plan:internal/loomshed#patternChangesMismatchCheck` -> `const patternChangesMismatchCheck = "pattern-changes-mismatch"`
- `plan:internal/loomshed#seedPlanContextFixture` -> `func seedPlanContextFixture(t *testing.T, anchorPath, renamePair, planSection, recordSection string) PlanContext`
- `plan:internal/shedrecipe#planContextOf` -> `func planContextOf(env Env) loomshed.PlanContext`

**Edit:**
- `internal/loomshed#ValidatePlan` -> `func ValidatePlan(plan *planparser.Plan, anchorPath, worktreeRoot string, index planindex.Index, pc PlanContext) ([]planindex.Finding, error)`
- `internal/loomshed#ValidatePlanLensReview` -> `func ValidatePlanLensReview(plan *planparser.Plan, anchorPath, worktreeRoot string, index planindex.Index, pc PlanContext) ([]planindex.Finding, error)`
- `internal/loomshed#ValidateReworkPlan` -> `func ValidateReworkPlan(plan *planparser.Plan, worktreeRoot string, index planindex.Index, readCommitted func(anchorRel string) ([]byte, bool, error), pc PlanContext) ([]planindex.Finding, error)`
- `internal/loomshed#NewPlanGate` -> `func NewPlanGate(anchorPath, worktreeRoot, scratchDir string, index planindex.Index, pc PlanContext) shuttleengine.Gate`
- `internal/loomshed#NewLensReviewPlanGate` -> `func NewLensReviewPlanGate(anchorPath, worktreeRoot, scratchDir string, index planindex.Index, pc PlanContext) shuttleengine.Gate`
- `internal/loomshed#NewReworkPlanGate` -> `func NewReworkPlanGate(anchorPath, worktreeRoot, scratchDir string, index planindex.Index, readCommitted func(anchorRel string) ([]byte, bool, error), pc PlanContext) shuttleengine.Gate`
- `internal/loomshed/doc.go`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#resolvePlainGateClosure`
- `internal/shedrecipe#planWriteGateSpec`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#loomCLI.validatePlanCmd`
- `internal/loomcli#loomCLI.wireLightweight`
- `internal/loomshed#TestNewPlanGate`
- `internal/loomshed#TestNewPlanGate_ParsePlanSplit`
- `internal/loomshed#TestNewReworkPlanGate`
- `internal/loomshed#TestPlanGate_FailureSurfacesItsFindings`
- `internal/loomcli#TestValidatePlanCmd_ReworkReportsGateFindingsForFirstCardMismatch`
- `internal/loomcli#TestGateParity_PlanGate`
- `internal/loomcli#TestGateParity_ReworkPlanGate`
- `internal/loomcli#TestWire_LanguageSkills`
- `internal/loomcli#TestValidatePlanCmd`
- `internal/shedrecipe#TestResolveGateSpec_ValidOneEntryPerVocabularyName`
- `internal/shedrecipe#TestPlanWriteEntry_GateConfig`
- `contracts/specs/loom-plan-spec.md`
- `contracts/stencils/loom/loom-template-plan.md`

**Uses:**
- `plan:internal/planparser#SymbolRenamePairs`
- `plan:internal/planparser#PatternChange`
- `plan:internal/discussionparser#ParsePatternChanges`
- `internal/languageconfig#Load`
- `internal/loomshed#seedPlanFormatFixture`

**Intent:**
D1.7 and D5.3: two plan-gate checks outside `internal/planparser`, in the one loomshed function set the Plan-Write and Plan-Burler-Fix gate closures and `lyx loom validate-plan` share (`PATTERN-gate-self-check-parity`), in the default, lens-review and rework modes alike.

- `PlanContext` holds `RenameTools []string`, the hub's `language.yaml` list, and `DecisionRecordPath string`, the record whose `## PATTERN changes` the plan must carry, empty to skip that check.
- `planContextFindings` returns, in this order:
  - with `pc.RenameTools` empty, one blocking `rename-tool-missing` finding per pair `planparser.SymbolRenamePairs(plan, cards)` returns, its `Card` the pair's card and its detail naming the pair and "way forward: set rename_tools with \"lyx config language\" from the prime, or recast the card"; a file pair and a non-empty list yield none;
  - with `pc.DecisionRecordPath` set and the record present, one blocking `pattern-changes-mismatch` finding per record entry (ID, action, text) the plan's `PatternChanges` lacks and per plan entry the record lacks, each detail naming the bullet and "way forward: copy the decision record's `## PATTERN changes` bullets into `00-overview.md`'s `## PATTERN changes` section, one per entry, without the added-after-Discussion suffix"; both lists empty is clean.
    An absent record file adds nothing; a read error is a returned error; a record whose section `ParsePatternChanges` flags is a returned error naming the findings and `lyx loom validate-discussion`, since the plan writer cannot fix the record.
- `ValidatePlan` appends `planContextFindings` over every card not in `websterengine.DoneCards`, because a done card's rename already ran; `ValidatePlanLensReview` passes `pc` through; `ValidateReworkPlan` appends it over every card of the fresh generation.
  The three gate constructors pass `pc` to their validator.
- `shedrecipe.Env` gains `RenameTools []string`; `planContextOf` builds `PlanContext{RenameTools: env.RenameTools, DecisionRecordPath: env.PatternRecordPath}`, which `resolvePlainGateClosure`'s `plan` and `rework-plan` cases and `planWriteGateSpec` pass.
  `env.PatternRecordPath` is card 10's loom-only record path, so a darn run is not checked against a record, as D5.3 says.
  `loomCLI.wire` sets `c.env.RenameTools` from the `language.yaml` it already loads.
- `validatePlanCmd`, on the lightweight wiring that loads no config, loads `languageconfig.Load(fabricengine.ConfigDirs(c.location))` itself in its default, `--lens-review` and `--rework` modes, refusing with the loader's error (which names `lyx config language`) when it fails, and builds `PlanContext` from it and `c.env.PatternRecordPath`.
- `wireLightweight`'s doc comment, which says `validateDiscussionCmd` and `validatePlanCmd` read only paths and need no config load, now says that `validate-plan` loads the hub-wide `language.yaml` itself for its `rename-tool-missing` check, and that this path still loads none of `wire()`'s module configs.
- `loom-plan-spec.md`'s validation-checks preamble lists `rename-tool-missing` and `pattern-changes-mismatch` beside `done-card-edited` as checks `loomshed` appends outside the `planparser` entry points, in every mode, with their conditions and ways forward above; `loom-template-plan.md` tells the writer to copy the record's `## PATTERN changes` bullets into the overview's section when the record has one, and that a symbol `Rename` needs the hub's `rename_tools`.
- `doc.go` names both checks and `PlanContext`.

Tests:
`seedPlanContextFixture` writes a one-card plan under `anchorPath` whose card renames by `renamePair` (empty for an `Edit` card), with `planSection` as its overview's `## PATTERN changes` body, writes a decision record with `recordSection` unless `recordSection` is the literal `absent` (empty writes a record with no section), and returns the `PlanContext` naming the record path.
`TestNewPlanGate` gains rows calling `ValidatePlan` and counting each new check ID in the findings:
- a symbol pair under an empty list yields one `rename-tool-missing`, and none with a non-empty list; a file pair under an empty list yields none;
- the same symbol-pair card recorded done in webster's run record (a `state.json` under the anchor whose batch holds the card done, as the existing done-card rows of this test seed it) yields no `rename-tool-missing`;
- a record section the plan lacks, a plan section the record lacks, an entry whose text differs, an entry whose action differs with the same ID, and a plan line that fits no bullet shape each yield `pattern-changes-mismatch`;
- matching sections, two absent sections and a `PlanContext` naming a record path with no file there yield none;
- a malformed record section is a returned error naming `lyx loom validate-discussion`;
- a `NewLensReviewPlanGate` row shows the lens mode reports `rename-tool-missing` too.

`TestNewReworkPlanGate` gains the symbol-pair row in rework mode.
`TestNewPlanGate_ParsePlanSplit` and `TestPlanGate_FailureSurfacesItsFindings` pass a zero `PlanContext`.
`TestGateParity_PlanGate` gains a row whose plan holds a symbol pair under an empty list, reported identically by the gate and the verb, and a row whose hub `language.yaml` lists a rename tool, where neither reports `rename-tool-missing`, which shows the verb loads the file.
`TestGateParity_ReworkPlanGate` gains the symbol-pair row under an empty list, reported identically by the rework gate and `validate-plan --rework`.
`TestValidatePlanCmd_ReworkReportsGateFindingsForFirstCardMismatch` keeps its case with no `language.yaml`; `TestValidatePlanCmd` gains a row whose hub `language.yaml` holds an empty `rename_tools` entry, refused naming `lyx config language`, and a `--lens-review` row over a symbol-pair plan reporting `rename-tool-missing`.
`TestWire_LanguageSkills` asserts `c.env.RenameTools` equals the file's list.
`TestResolveGateSpec_ValidOneEntryPerVocabularyName`'s `plan` and `rework-plan` entries and `TestPlanWriteEntry_GateConfig` each gain a case whose env carries `RenameTools: nil` over a seeded symbol-pair plan: the constructed gate fails naming `rename-tool-missing`, and with `RenameTools` set it does not, so a site passing a zero `PlanContext` fails the case.

**ImpactSummary:** six re-signed loomshed functions whose production callers are the three recipe gate sites and `validate-plan`; a plan without a symbol `Rename` pair and without a record section validates as before.
