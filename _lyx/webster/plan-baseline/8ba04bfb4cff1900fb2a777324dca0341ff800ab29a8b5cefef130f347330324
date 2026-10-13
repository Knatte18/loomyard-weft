# Card 7 — twill-final-self-check

**Edit:**
- `//contracts/stencils/twill/twill-directive.md`
- `contracts/stencils#TestStencils_TopLevelMarkers`
- `contracts/stencils#wordingClaims`
- `internal/twilldirective#Values`
- `internal/twilldirective#Directive`
- `internal/twilldirective#filledValues`
- `internal/twilldirective#TestDirective_RendersEveryValueVerbatim`
- `internal/twilldirective#TestDirective_MissingValueNamesTheMarker`
- `internal/loomengine#planTwillDirective`
- `internal/loomengine#twillBlocks`
- `internal/loomengine#TestPlanSpec_Twill`
- `internal/loomengine#TestDiscussionSpec_Twill`
- `internal/loomengine/plan.go`
- `//docs/overview.md`

**Intent:**
D4: the twill directive's final self-check, step 8, takes its own host value, so the twill Plan writer runs `lyx loom validate-plan --lens-review` (card 6) last and meets a missing lens review in its own turn instead of in a gate re-prompt.
The pre-ready self-check, step 3, keeps `lyx loom validate-plan`, since it runs before the forks return and cannot require a file written only after them.
The Discussion host passes `lyx loom validate-discussion` for both, so its rendered prompt is unchanged.

In `twill-directive.md`, step 8 becomes ``8. Run `{{.final_self_check_command}}` again until it passes, then end your turn.``, and the banner's marker list gains `{{.final_self_check_command}}`; step 3 keeps `{{.self_check_command}}`.
The stencil stays host-neutral: the command arrives through the marker.

`twilldirective.Values` gains `FinalSelfCheckCommand string`, documented as the command the writer runs as the final self-check, after the lens review is recorded; `Directive` fills the marker `final_self_check_command` from it, required like every other marker.
`planTwillDirective` sets it to `lyx loom validate-plan --lens-review`, and `twillBlocks` to `lyx loom validate-discussion`.
These two constants are rendered here only; this card does not depend on whether the forks spawn before or after the files are written, so it holds under either spawn point the board's `twill-fork-timing` task sets.

Tests:
- `TestStencils_TopLevelMarkers`' `twill-directive` row gains `final_self_check_command` in its sorted marker list.
- `wordingClaims`' `twill-directive.md` entry gains the claim `{must: "{{.final_self_check_command}}", why: "the host names the final self-check command"}`.
- `filledValues` sets `FinalSelfCheckCommand: "lyx check --final"`; `TestDirective_RendersEveryValueVerbatim` wants ``8. Run `lyx check --final` again`` and ``run `lyx check --now` per `STEP-CHECK` until it passes``; `TestDirective_MissingValueNamesTheMarker` gains the row `{"FinalSelfCheckCommand", func(v *Values) { v.FinalSelfCheckCommand = "" }, "final_self_check_command"}`.
- `TestPlanSpec_Twill` wants ``run `lyx loom validate-plan` per `Step 5` until it passes`` and ``8. Run `lyx loom validate-plan --lens-review` again until it passes``.
- `TestDiscussionSpec_Twill` wants ``run `lyx loom validate-discussion` per `Step 6` until it passes`` and ``8. Run `lyx loom validate-discussion` again until it passes``, pinning the Discussion prompt's steps unchanged.

Docs: `plan.go`'s file comment on `plan_producer` twill gains that the writer runs `lyx loom validate-plan --lens-review` as its final self-check and that its row's gate requires a recorded `lens-review.md` for a twill-spawned run.
`docs/overview.md`'s twill plan writer sentence (the one beginning "`plan_producer: twill` selects the single plan writer") gains that it runs `validate-plan --lens-review` as its last self-check and that its gate refuses Done until `lens-review.md` holds the lens review, and the following sentence, "The row keeps its name, gates and routing whichever producer runs.", says the row keeps its name, `gates` key and routing while a twill-spawned run's plan gate also requires the lens review.

**ImpactSummary:** one new required marker in one host-neutral stencil with exactly two renderers, both updated here; the Discussion prompt's bytes are unchanged, and the Plan prompt changes in step 8 only.

**Commit:** 7: twill: the Plan writer's final self-check requires the lens review
