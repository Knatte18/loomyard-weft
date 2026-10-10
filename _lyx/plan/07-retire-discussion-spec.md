# Card 7 — retire-discussion-spec

**Delete:**
- `internal/loomengine#DiscussionSpec`
- `internal/loomengine/prompt.go`
- `internal/loomengine#TestDiscussionSpec`
- `internal/loomengine#TestDiscussionSpec_Refuses`
- `internal/loomengine#TestDiscussionSpec_ReadsStencilAtCallTime`
- `internal/loomengine#TestDiscussionSpec_PatternDirective`
- `internal/loomengine#TestComposePrompt_RendersMarkers`

**Edit:**
- `internal/loomengine#TestProducerSpecs_SkillsAndParentDirective`
- `internal/loomengine#TestCompose_FrictionDirective`
- `internal/mergeresolve/spec.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/loomengine#DiscussionTable`
- `plan:internal/seatengine#Prompts`

**Intent:** Remove the single-agent discussion path (D1) now that nothing references it.
`DiscussionSpec` leaves `discussion.go`, whose header comment then describes the role and skill constants it keeps; `prompt.go` is deleted whole with `composePrompt` and `modeRules`, whose callers were `DiscussionSpec` and the two tests this card deletes or rewrites.
The four `DiscussionSpec` tests are deleted, their behaviours being rows of card 5's `TestDiscussionTable` and `TestDiscussionTable_Refuses`; `TestComposePrompt_RendersMarkers` is deleted for the same reason.
`TestProducerSpecs_SkillsAndParentDirective`'s two discussion rows build the table through `DiscussionTable` and read the chair's skills from the chair seat and the chair prompt from `seatengine.Prompts` with the row's parent name, keeping the assertion that the operator-ban line is absent only for the interactive chair; the plan and rework rows are unchanged.
`TestCompose_FrictionDirective`'s discussion row composes the chair prompt through `DiscussionTable` and `Prompts`: the enabled case sets `cfg.Friction` and asserts the note path under the layout's friction directory lands verbatim, the disabled case leaves it empty over the minimal stencils directory, and the marker-free case rewrites the chair stencil without the friction marker and still composes; the plan row is unchanged.
The header comments of `discussion_test.go` and `prompt_test.go` name what each file still holds.
`mergeresolve/spec.go`'s two comments that cite `loomengine.DiscussionSpec` as the resolution shape cite `loomengine.PlanSpec`, which keeps that shape.
The budget file's `internal/loomengine` row is set to the package's top-level test count after this card.

**ImpactSummary:** Deletes one exported function and one file with no remaining callers; two loomengine tests are rewritten and one comment file corrected, with no behaviour change outside tests.
