# Fixer report — plan, round 1

## Changes

- **F1** (fixed as proposed): card 5 lists `internal/shedbuild#TestBuild_EveryRegisteredEngineBuilds` under Edit after `TestBuild_Errors`, and its Tests gain a bullet: the one-row recipe sets `BlockGate: shedrecipe.BlockGateAlways` on the `shedrecipe.EngineBlock` row.
- **F2** (fixed as proposed, then widened by the self-check): card 3 lists `contracts/specs/shed-recipe-spec.md` under Edit, with an Intent line for the Document shape paragraph's `profile_recipe` key, the block row and the rewritten row-key sentence.
  Card 4 lists the spec too, under Edit rather than the proposed Extends, since the self-check found two existing paragraphs that its change rewrites (see Self-check).
  Its Intent line adds the expansion paragraph and rewrites the exported-function and Validation split paragraphs.
  Card 5 also lists the spec, so the exported-function paragraph names `RoutingFrom` and the expanded-recipe rule.
- **F3** (fixed as proposed): card 6 lists `internal/loomrecipe#TestInterruptPolicies_MatchAssembledRows` and `internal/loomshed#InterruptPolicies` under Edit after `TestRecipeKeys`.
  Its Intent maps `NameMotif` to `shedverbs.InterruptPolicyReinvoke`, and its Tests add `shedrun.RecipeArras` to the pin's recipes.
- **F5** (fixed as proposed, then widened): card 7 moves `internal/battenshed/doc.go` from Uses to Edit and adds `docs/overview.md`.
  The Intent names arras in battenshed's escalate-halt sentence and in overview's loomrecipe line, arming-table line and `lym loom` verbs list, and names motif as a block source.
  The fix also covers three more overview sentences that list the recipes: the `EscalateStopPrefix` escalate halt, the batten bullet's child recipes, and the runs `Run-Shed` seeds.
  The self-check's wider set is recorded below.
- **F6** (fixed as proposed): card 6's TestBlockRecipes keeps only the build clause of the ring bullet and drops the New-versus-Routing bullet.
- **F7** (fixed as proposed): card 7 drops `internal/loomcli#TestWire_DefaultConfig` from Edit, `internal/loomcli#loomSegmentPerches` from Uses, and the arras budget-pass test bullet.

## Disputed

- **F4**, ground `premise false`.
  The finding says Seed-Child's `WriteSeed` runs before `writeRunSettings`, so no recipe that fails `ProfileRecipe` reaches `prepareRunSettings`.
  The code runs them in the opposite order.
  `internal/battenshed/seamchild.go:74` calls `p.writeRunSettings(ctx, recipe)` straight after `ReadBoardType`, and that call reaches `PrepareRunSettings` at `:141`.
  `p.deps.WriteSeed` is called only later, at `:93`.
  `Call`'s doc at `:48` states the same order: "It then writes the task worktree's local config overrides from the Board task's run settings, reads the driver after them …, then writes, commits and pushes the seed".
  So a task whose recipe names no embedded recipe does reach `prepareRunSettings` before `WriteSeed`'s `ValidateRecipe` and `childRecipeSupported` checks (`internal/battencli/wire.go:1392-1401`).
  Card 8's fallback is reachable, its sentence that such a task reaches Seed-Child's seed write is correct, and its `TestPrepareRunSettings` row covers a real path.
  Card 8 is unchanged.

## Deferred

None.

## Self-check

Every fork returned a report.

- fix-claims: no findings.
- fix-proposal, MEDIUM: card 4's spec paragraph contradicts the spec's Validation split paragraph. Fixed as proposed: card 4 rewrites that paragraph to name `Expand`'s block refusals as the exception, so the spec moves from Extends to Edit.
- fix-proposal, LOW: the "Four exported functions" paragraph goes stale. Merged with fix-consistency's third finding and fixed as proposed: card 4 names `Expand` there and drops the count, and card 5 names `RoutingFrom`.
- fix-consistency, LOW: the doc comment and error text of `TestInterruptPolicies_MatchAssembledRows` still name three recipes. Fixed as proposed in card 6's Tests bullet.
- fix-consistency, LOW: card 6's ImpactSummary omits the `InterruptPolicies` entry. Fixed as proposed.
- fix-consistency, LOW: the spec's Four-functions and Validation split paragraphs, and card 5's `RoutingFrom` and expanded-recipe rule, are missing. Merged with fix-proposal's two findings; card 5 now lists the spec.
- fix-affected-set, MEDIUM: many more comments, specs, stencils and two user-facing refusals list the loom recipes without arras. Fixed as proposed: card 7's Edit gains each file, and its Intent gains two bullets.
  The first covers boardengine's project-recipe refusal and driftcli's refusal, help text and doc.
  The second covers the remaining comment, spec, stencil and `orch-role.md` sentences and gateslot's slot-table rows.
  The ImpactSummary now counts three refusal texts.
  The fixer's own grep found the driftcli and `orch-role.md` members, which the fork did not list.
  The plan gate then found that some of those entries were whole files that overlap member glyphs or seam types.
  Card 7 now lists `internal/landingshed#Deps` and `internal/battenshed#InnerRunDeps` instead of their `deps.go` files.
  It lists `internal/battencli#battenCLI.wire` and `internal/battencli#restepSkips` instead of `internal/battencli/wire.go`.
  Its Uses gains `internal/boardengine/doc.go`, which needs no change, and the seam tests that reflect over the two Deps types.
  Those tests are `battencli#TestWire_EverySeamFilled`, `TestLandingDeps_EveryFieldPopulated`, loomcli's `TestWire_BeamRecipe`, `TestWire_DarnRecipe`, `TestWire_EverySeamFilled` and `TestWire_SamplerRecipe`, and envkit's `TestFullEnv_FillsEverySeam` and both `TestNilSeams` tests; the card changes only comments, so they stay as they are.
  It left out `internal/lenses/config.go`, which names fan config keys rather than the accepted recipes, and `loomrecipe`'s `finalizerow.go` and `bouncerrows.go`, which read only tapestry and darn.
