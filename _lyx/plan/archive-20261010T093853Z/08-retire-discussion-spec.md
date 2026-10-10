# Card 8 — retire-discussion-spec

**Delete:**
- `internal/loomengine#DiscussionSpec`
- `internal/loomengine/prompt.go`
- `internal/loomengine#TestDiscussionSpec`
- `internal/loomengine#TestDiscussionSpec_Refuses`
- `internal/loomengine#TestDiscussionSpec_ReadsStencilAtCallTime`
- `internal/loomengine#TestDiscussionSpec_PatternDirective`
- `internal/loomengine#TestComposePrompt_RendersMarkers`

**Edit:**
- `contracts/stencils/loom/loom-template-discussion.md`
- `plan:internal/loomengine#TestDiscussionTable`
- `internal/loomengine#TestProducerSpecs_SkillsAndParentDirective`
- `internal/loomengine#TestCompose_FrictionDirective`
- `internal/loomengine/review.go`
- `internal/loomengine#PlanSpec`
- `internal/shedadapters#SpecSource`
- `internal/mergeresolve/spec.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/loomengine#DiscussionTable`
- `plan:internal/seatengine#Prompts`

**Intent:** Remove the single-agent discussion path (D1) now that nothing references it, and give the chair stencil the chair seat directive, which that path's composer could not fill (D6, D8).
`DiscussionSpec` leaves `discussion.go`, whose header comment then describes the role and skill constants it keeps; `prompt.go` is deleted whole with `composePrompt` and `modeRules`, whose callers were `DiscussionSpec` and the two tests this card deletes or rewrites.
With that composer gone, the chair stencil gains the include `{{template "seat-directive-chair"}}` after the edit directive, and its header comment drops `composePrompt` and `DiscussionSpec`, naming the seat engine as its only filler; the seat engine fills the block's markers, so the chair prompt names every advisor strand, the advisors that never started and the advisor notes files as inputs.
A row joins `TestDiscussionTable` asserting, through `seatengine.Prompts`, that the chair prompt names every advisor's strand name and notes path for one and two advisors, and that with zero advisors it still composes.
The four `DiscussionSpec` tests are deleted, their behaviours being rows of card 5's `TestDiscussionTable` and `TestDiscussionTable_Refuses`; `TestComposePrompt_RendersMarkers` is deleted for the same reason.
`TestProducerSpecs_SkillsAndParentDirective`'s two discussion rows build the table through `DiscussionTable` and read the chair's skills from the chair seat and the chair prompt from `seatengine.Prompts` with the row's parent name, keeping the assertion that the operator-ban line is absent only for the interactive chair; the plan and rework rows are unchanged.
`TestCompose_FrictionDirective`'s discussion row composes the chair prompt through `DiscussionTable` and `Prompts`: the enabled case sets `cfg.Friction` and asserts the note path under the layout's friction directory lands verbatim, the disabled case leaves it empty over the minimal stencils directory, and the marker-free case rewrites the chair stencil without the friction marker and still composes; the plan row is unchanged.
The header comments of `discussion_test.go` and `prompt_test.go` name what each file still holds.
Comments that cite the retired `DiscussionSpec` as a live sibling or shape are re-pointed: `review.go`'s file header contrasts `ResolveReview` with `PlanSpec` alone, `PlanSpec`'s doc calls it a pure composer without the comparison, `SpecSource`'s doc drops its sentence naming `loomengine.DiscussionSpec`, and `mergeresolve/spec.go`'s two comments cite `loomengine.PlanSpec`, which keeps that shape.
The budget file's `internal/loomengine` row is set to the package's top-level test count after this card.

**ImpactSummary:** Deletes one exported function and one file with no remaining callers; the chair stencil's one include is filled by the seat engine, the row's only composer after card 7, two loomengine tests are rewritten, one gains a row, and four files' comments are corrected, with no behaviour change outside the chair prompt and tests.
