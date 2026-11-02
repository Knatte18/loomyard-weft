# Card 10 — writer-stencils-fixer-mode

**Create:**
- `contracts/stencils/twill/twill-writer-fork-gated.md`
- `plan:contracts/stencils#TwillWriterForkGated` -> `var TwillWriterForkGated []byte`
- `contracts/stencils/loom/loom-fragment-self-check-plan.md`
- `contracts/stencils/loom/loom-fragment-self-check-plan-gated.md`
- `contracts/stencils/loom/loom-fragment-self-check-discussion.md`
- `contracts/stencils/loom/loom-fragment-self-check-discussion-gated.md`
- `plan:contracts/stencils#LoomFragmentSelfCheckPlan` -> `var LoomFragmentSelfCheckPlan []byte`
- `plan:contracts/stencils#LoomFragmentSelfCheckPlanGated` -> `var LoomFragmentSelfCheckPlanGated []byte`
- `plan:contracts/stencils#LoomFragmentSelfCheckDiscussion` -> `var LoomFragmentSelfCheckDiscussion []byte`
- `plan:contracts/stencils#LoomFragmentSelfCheckDiscussionGated` -> `var LoomFragmentSelfCheckDiscussionGated []byte`
- `plan:internal/loomengine#planTwillSpawnPointGated` -> `const planTwillSpawnPointGated = ""`
- `plan:internal/loomrecipe#GateFixerSetting` -> `func GateFixerSetting(recipe, engine, gate string) (string, error)`
- `plan:internal/loomrecipe#TestGateFixerSetting` -> `func TestGateFixerSetting(t *testing.T)`
- `plan:internal/loomcli#writerFixerOn` -> `func writerFixerOn(recipe, engine, gate string, setting func(module, key string) (any, error), pairSlug string) (bool, error)`
- `plan:internal/loomcli#TestWriterFixerOn` -> `func TestWriterFixerOn(t *testing.T)`

**Edit:**
- `internal/twilldirective#Values`
- `internal/twilldirective#Directive`
- `contracts/stencils/twill/twill-directive.md`
- `internal/twilldirective#TestDirective_RendersEveryValueVerbatim`
- `contracts/stencils/loom/loom-template-plan.md`
- `contracts/stencils/loom/loom-template-discussion.md`
- `internal/loomengine#PlanTable` -> `func PlanTable(layout *lymcwd.Location, dirs configengine.Dirs, stencilsDir, specsDir string, cfg Config, reg modelspec.Registry, gateTiers string, fixerOn bool) (seatengine.Table, error)`
- `internal/loomengine#DiscussionTable` -> `func DiscussionTable(layout *lymcwd.Location, dirs configengine.Dirs, stencilsDir, specsDir string, cfg Config, reg modelspec.Registry, slug string, fixerOn bool) (seatengine.Table, error)`
- `internal/loomengine#planLensDirective` -> `func planLensDirective(layout *lymcwd.Location, dirs configengine.Dirs, stencilsDir, specsDir, fan string, fixerOn bool) (string, error)`
- `internal/loomengine#discussionLensDirective` -> `func discussionLensDirective(layout *lymcwd.Location, dirs configengine.Dirs, stencilsDir, specsDir, fan string, fixerOn bool) (string, error)`
- `internal/loomengine#TestPlanTable`
- `internal/loomengine#TestDiscussionTable`
- `internal/loomengine#TestDiscussionTable_Refuses`
- `internal/loomengine#TestProducerSpecs_SkillsAndParentDirective`
- `internal/loomcli#loomCLI.wire`
- `contracts/stencils#TestStencils_TopLevelMarkers`

**Extends:**
- `contracts/stencils#entries`
- `internal/twilldirective/doc.go`
- `internal/loomrecipe/doc.go`

**Uses:**
- `plan:internal/loomcli#pairSlugOf`
- `internal/loomcli/doc.go`
- `internal/twilldirective#filledValues`
- `contracts/stencils#TestTwillStencils_HostNeutral`

**Intent:**
With the fixer on, the Plan-Write and Discussion-Write writers, the Discussion chair included, stop looping on their own validator (D7): they end their turn once their files are written and the row gate and its fixer check them; with the fixer off, or in a worktree with no pair slug, every prompt renders today's text.

- `loomrecipe.GateFixerSetting(recipe, engine, gate)` returns the `fixer` value of the `gate` element of the named recipe's row whose engine is `engine`, `""` when the element carries none; an unknown recipe, or a recipe with no row of that engine, is refused naming it, as `EngineRow` refuses.
- `writerFixerOn(recipe, engine, gate, setting, pairSlug)` is true only when the pair slug is non-empty, the element names a fixer setting and `setting` holds a non-empty value for it, the three conditions under which card 8's seam wraps the gate; a setting lookup error is returned.
- `loomCLI.wire` computes it for `PlanWrite`'s `plan` and `DiscussionWrite`'s `discussion` with `Env.Setting` and `pairSlugOf`, inside the `PlanTable` and `DiscussionTable` closures so the stencils and settings stay read at call time, and passes it to `PlanTable` and `DiscussionTable`.
- `PlanTable` and `DiscussionTable` take `fixerOn` and fill a new `{{.self_check}}` marker in `loom-template-plan.md`'s Step 5 and `loom-template-discussion.md`'s Step 6 from `loom-fragment-self-check-plan` or `-plan-gated`, `loom-fragment-self-check-discussion` or `-discussion-gated`.
  The ungated fragments hold today's step text verbatim; the gated ones say the files are checked by the row's gate once the turn ends, whose cold fixer clears mechanical findings first, so the writer runs no validator and ends its turn once its files are written, its lens review recorded where it runs one.
  They pass `fixerOn` to `planLensDirective` and `discussionLensDirective`.
- `twilldirective.Values` gains `Gated bool`.
  With it set, `Directive` fills the writer-fork part from `twill-writer-fork-gated` instead of `twill-writer-fork`, fills the closing step `{{.closing_step}}` with the line that ends the turn without a validator, and leaves `{{.checked_clause}}` empty; unset, `{{.closing_step}}` is today's step 6 over `final_self_check_command` and `{{.checked_clause}}` is ` and passed the self-check`, so the ungated directive renders today's text byte for byte.
  `SelfCheckCommand` and `FinalSelfCheckCommand` may be empty only with `Gated` set; otherwise every marker stays required.
- `twill-writer-fork-gated.md` is `twill-writer-fork.md` without the validator: fork0 writes the files, the last file last, and returns naming the files it wrote; step d reads them, and the first resume rule completes missing files per the format step only.
  It names no host, as `TestTwillStencils_HostNeutral` requires.
- `planLensDirective` and `discussionLensDirective` set `Gated` from `fixerOn`; the plan host takes `planTwillSpawnPointGated` in place of `planTwillSpawnPoint`, the same opening without "and `lym loom validate-plan` passes", and the discussion host's spawn point drops "and `lym loom validate-discussion` passes per Step 6", so the writer runs its lens review right after writing.
- The new stencils are embedded and registered, with their marker rows in `TestStencils_TopLevelMarkers`, and the directive's and both templates' rows gain their new markers.

Bound: with the fixer on, a fixer may edit files the writer's lens review read; that staleness is accepted (D7), the plan gate checks the lens review record for presence only, and D5's later rows judge the fixer's edits.

Tests, tier0:
- `TestGateFixerSetting`: the shipped tapestry recipe's `PlanWrite` `plan` element names `tapestry.plan_gate_fixer` and its `DiscussionWrite` `discussion` element `loom.discussion_gate_fixer`; the darn recipe's `DarnWrite` `verify` element names `darn.gate_fixer`; an element with no `fixer` gives `""`; an unknown recipe is refused.
- `TestWriterFixerOn`: on with a slug, a named setting and a non-empty value; off with an empty slug whatever the setting; off with an empty value; off for an element with no `fixer`; a lookup error is returned.
- `TestDirective_RendersEveryValueVerbatim` gains a gated row: the gated writer-fork part, no `final_self_check_command` and no "passed the self-check"; the ungated rows stay byte-for-byte.
- `TestPlanTable` and `TestDiscussionTable` gain rows: with `fixerOn` true the lead's prompt carries the gated self-check fragment and the gated spawn point, and lacks the three loop lines: the self-check step's command block, the spawn point's "`lym loom validate-plan` passes per" (or "`lym loom validate-discussion` passes per Step 6") clause and the closing "again until it passes" step; the dispute-evidence phrase naming `lym loom validate-plan` output stays, so no row asserts that string absent; with `false` it renders today's prompt; every existing call passes `false`.
- `TestDiscussionTable` also gains a row with advisors and `fixerOn` true: the chair seat's prompt carries the gated fragment and lacks the loop lines, since the chair template wraps the discussion template.
- `TestDiscussionTable_Refuses` and `TestProducerSpecs_SkillsAndParentDirective` pass `false` and keep their assertions.

**ImpactSummary:** two table renderers and their two directive helpers are re-signed, with every caller in `internal/loomengine`'s tests and `loomCLI.wire`; with `fixerOn` false, which every non-pair run and every hub with an empty fixer key gets, the prompts are unchanged.
