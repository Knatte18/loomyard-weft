```yaml
lens: affected-set
outcome: reported
```

Lens: affected-set. For each output, key, string, file shape or behaviour a card changes, I found what reads or asserts it and which cards cover it.

**Per change: the set I found, and the cards that cover it**

- **`planGate` signature change (card 7).** Callers: `NewPlanGate` (gates.go:236) and `NewLensReviewPlanGate` (gates.go:298). Card 7 covers both.
- **`settledSegmentGuard` and `segmentWriter` signature change (card 20).** Readers:
  - `loomCLI.settledGuard` (settledguard.go:21) and the `segmentWriter` call inside the guard (settledguard.go:60).
  - `TestSettledSegmentGuard` (settledguard_test.go:142).
  
  Card 20 covers all of them.
- **`refuseVerbOffRecipe` gaining the row-engine map (cards 18 and 19).** Its one caller is arm.go:257, which keeps its arguments. Cards 18 and 19 cover it, plus `TestArmAt_RefusesOffLoomRecipes`.
- **`Env` gaining four seams and `PartsMax` (card 16).** Readers:
  - production: loomcli `wiring.go`, `envkit.FullEnv`, battencli `intentionallyNil` and `TestWire_EverySeamFilled`;
  - the envkit reflective tests;
  - other `shedrecipe.Env{}` literals in shedbuild, loomcli and battencli tests and in `landingdeps.go`, which are left zero.
  
  Card 16 covers the reflective set. The plain literals take zero values and need no edit.
- **`Seat.Required` now combined with the writer kind in `seatSpec` QuietHold (card 9).** Readers: `internal/seatengine/prompt_test.go` (edited by card 9 as `TestSeatPrompts_ValuesPromptsAndSpecs`). The shuttleengine QuietHold tests read the spec field, not the seat rule, so they are unaffected.
- **`Drift-Baseline` `on_done` becomes `Parts-Write`, and Plan-Write's position moves (card 17).** Readers of `NameDriftBaseline`:
  - `loomrecipe/shape_test.go` `wantProducerTable` (card 17);
  - `sequence_test.go` `deriveSequenceOrder` (card 17);
  - `drift_test.go`, which asserts no `on_done` (grep found none), so it is unaffected;
  - `reportengine/timing_test.go` `loomRowPhases` (card 17).
  
  Also `reviewbudget_test.go:133-140`, which asserts only `Plan-Write.OnDone == Plan-Bouncer`. That is unchanged, so it holds.
  
  Text readers:
  - `internal/loomrecipe/doc.go:15` (card 17);
  - `tapestry-recipe.yaml` header lines 9 and 23 (card 17);
  - `docs/overview.md:592` (card 17).
  
  Card 17 covers the full set.
- **Interrupt policy, timing phase and step test for the new row names (card 17).** `internal/loomcli/step_test.go:39` lists one reinvoke row as a sample, not every row, so it is unaffected. `TestInterruptPolicyFor` and `loomRowPhases` are covered by card 17.
- **Card-format and glyph text moved out of `loom-template-plan.md` into `loom-block-plan-format` (card 12).** Readers:
  - `TestStencils_TopLevelMarkers` (card 12);
  - `TestHubGenericStencils_NameNoGoTiers` (govalues_test.go:103) (card 12);
  - `TestPlanTable` and its golden (card 12);
  - `TestStencils_NameGoValues`, which does not list `loom-template-plan`, so it is unaffected;
  - `TestLoomRubrics`, which pins only that the plan-review rubric names the template's path, which still exists, so it holds;
  - `stencilstore_test.go:194`, a path-only reader, unaffected.
  
  Prose readers that say the card format is pinned in `loom-template-plan.md`:
  - `contracts/stencils/loom/loom-rubric-plan-review.md:121`;
  - `contracts/specs/loom-plan-spec.md:3, 7, 831`;
  - `contracts/specs/webster-spec.md:14, 73`;
  - `docs/overview.md:422`.
  
  No card covers these.

**Findings**

1. **MEDIUM, consistency.** Location: card 12 (`12-plan-format-block.md`).
   Summary: card 12 moves the card format out of `loom-template-plan.md` but leaves every doc that says the format is pinned in that stencil. It covers the test readers of the moved text and misses the prose readers.
   - Missed members:
     - `contracts/specs/loom-plan-spec.md:3`, `:7` and `:831`: "the LLM-facing subset … is pinned separately in … `loom-template-plan.md`";
     - `contracts/specs/webster-spec.md:14` and `:73`;
     - `docs/overview.md:422`;
     - `contracts/stencils/loom/loom-rubric-plan-review.md:121`.
   - Why the rubric line matters: it is the writer/reviewer symmetry pointer, "The plan writer's own stencil is `…/loom-template-plan.md`". Once the move lands, a reviewer opening that file sees only `{{template "loom-block-plan-format"}}` where the card-format rules were.
   - Scope limit: the rubric line is Plan-Review's, and the record's Scope puts "Changes to … Plan-Review" out. A fix there must stay a pointer edit, or the gap should be recorded as a known stale pointer. The spec and overview lines carry no such limit.
   - Evidence: the grep results at those lines; card 12's Edit list holds only `loom-template-plan.md` and three tests.

**Tool calls**

1. Greps for callers of `planGate`, `settledSegmentGuard`/`segmentWriter` and `refuseVerbOffRecipe`, for `shedrecipe.Env{}` builders, and for QuietHold tests. The manifest read only the definitions.
2. Greps for users of `NameDriftBaseline` and for text tying `Drift-Baseline` to Plan-Write. Needed to find the edge's readers.
3. Grep of drift, sequence and other tests for `OnDone`/`NamePlanWrite`. Needed to see which tests assert the moved edge.
4. Read of the `reviewbudget_test.go` edge map, plus a grep for tests and docs that read `loom-template-plan` content. Needed to find card 12's readers.
5. Read of `rubric_test.go` and `govalues_test.go`. Needed to see whether either asserts content of the moved template.
6. Grep for docs and specs that cite `loom-template-plan.md` as the format's home. Needed to find card 12's prose readers.
