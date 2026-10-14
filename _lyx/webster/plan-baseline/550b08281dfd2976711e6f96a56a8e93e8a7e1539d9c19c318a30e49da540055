# Card 18 — retired-builders-delete

**Delete:**
- `internal/loomengine#DiscussionSpec`
- `internal/loomengine#composePrompt`
- `internal/loomengine#PlanSpec`
- `internal/loomengine#composePlanPrompt`
- `internal/loomengine#ReworkSpec`
- `internal/loomengine#composeReworkPrompt`
- `internal/landingshed#DescribeSpec`
- `internal/landingshed#DarnDescribeSpec`
- `internal/landingshed#describeSessionSpec`
- `internal/darnengine#DarnSpec`

**Edit:**
- `internal/landingshed#DescribeInputs`
- `internal/landingshed#DarnDescribeInputs`
- `internal/darnengine#DarnInputs`
- `internal/loomcli#loomCLI.wireDarn`
- `internal/loomcli#loomCLI.wire`
- `internal/loomengine#TestDiscussionTable`
- `plan:internal/loomengine#TestPlanTable`
- `plan:internal/loomengine#TestReworkTable`
- `internal/landingshed#TestDescribeSpec_ComposesSpec`
- `internal/landingshed#TestDarnDescribeSpec`
- `internal/darnengine#TestDarnSpec`
- `internal/landingshed/doc.go`
- `internal/darnengine/doc.go`
- `internal/loomengine/review.go`
- `//contracts/stencils/landing/landing-template-describe.md`
- `//contracts/stencils/landing/landing-template-describe-darn.md`
- `//contracts/stencils/darn/darn-template-write.md`

**Intent:** D10's removal of the single-writer spec builders, which cards 11 to 14 left uncalled: `DiscussionSpec` and `composePrompt`, `PlanSpec` and `composePlanPrompt`, `ReworkSpec` and `composeReworkPrompt`, `landingshed`'s `DescribeSpec`, `DarnDescribeSpec` and `describeSessionSpec`, and `darnengine.DarnSpec`, which card 15 left uncalled.
It runs after card 17, the last card that edits `loomCLI.wire`, so no later card edits code that names them.

The file headers that described them describe the table builders instead: `discussion.go` (keeping `DiscussionWriterRole`, `discussionRole` and `discussionSkills`), `prompt.go` (keeping `setOffParagraph` and `modeRules`), `plan.go`, `rework.go`, `landingshed/describe.go` and `darnengine/spec.go`.
`DescribeInputs`, `DarnDescribeInputs` and `DarnInputs` lose `ParentName`, which only the spec builders read, since the seat engine renders the parent directive; `loomCLI.wire`'s Describe closure and `loomCLI.wireDarn`'s two closures stop setting it.
The doc text that names a deleted builder names the table builder instead: `landingshed/doc.go`, `darnengine/doc.go`, `loomengine/review.go`'s header ("Unlike DiscussionSpec and PlanSpec"), and the header comments of the describe, darn-describe and darn-write stencils, including the darn stencil's note that the builder fills the two optional tokens with `(none)`.

The parity tests cards 11 to 15 wrote compare each table against the old spec builder; with the builders gone, `TestDiscussionTable`, `TestPlanTable`, `TestReworkTable`, `TestDescribeSpec_ComposesSpec`, `TestDarnDescribeSpec` and `TestDarnSpec` keep the same field assertions against the literal values the comparison pinned (role, model, effort, version, output files, skills, segment, timeout, interactivity, fork flag) and drop the byte comparison, whose prompts their substring rows still cover.
No other test calls a deleted builder, so these stand as the card's coverage.

**ImpactSummary:** Deletes uncalled builders in loomengine, landingshed and darnengine; three input types lose a field whose setters, all in loomcli wiring, are listed here.
