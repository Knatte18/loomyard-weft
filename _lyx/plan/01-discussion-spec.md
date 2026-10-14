# Card 1 — discussion-spec

**Create:**
- `contracts/specs/loom-discussion-spec.md`
- `contracts/specs#LoomDiscussionSpec`

**Edit:**
- `contracts/specs#entries`
- `contracts/specs#TestRegistry_NamesAndDefaults`
- `contracts/specs#TestRegistry_RelPathPlacesBothUnderLoomFamily`
- `contracts/stencils/loom/loom-template-discussion.md`
- `contracts/stencils/loom/loom-rubric-discussion-review.md`
- `contracts/stencils#wordingClaims`
- `internal/loomengine#composePrompt` -> `func composePrompt(stencilsDir, specsDir, slug, decisionRecordPath, supportLogPath, patternDirective, frictionDirective, parentDirective, twillDirective, twillWriteScope string, autonomous bool) ([]byte, error)`
- `internal/loomengine#DiscussionSpec` -> `func DiscussionSpec(layout *lyxcwd.Location, dirs configengine.Dirs, stencilsDir, specsDir, parentName string, cfg Config, reg modelspec.Registry, slug string, autonomous bool) (shuttleengine.Spec, error)`
- `internal/loomengine#DiscussionTable` -> `func DiscussionTable(layout *lyxcwd.Location, stencilsDir, specsDir string, cfg Config, reg modelspec.Registry, slug string) (seatengine.Table, error)`
- `internal/loomengine#TestComposePrompt_RendersMarkers`
- `internal/loomengine#TestCompose_FrictionDirective`
- `internal/loomengine#TestDiscussionSpec`
- `internal/loomengine#TestDiscussionSpec_Refuses`
- `internal/loomengine#TestDiscussionSpec_Twill`
- `internal/loomengine#TestDiscussionSpec_ReadsStencilAtCallTime`
- `internal/loomengine#TestDiscussionSpec_PatternDirective`
- `internal/loomengine#TestProducerSpecs_SkillsAndParentDirective`
- `internal/loomengine#TestDiscussionTable`
- `internal/loomengine#TestDiscussionTable_Refuses`
- `internal/loomcli#loomCLI.wire`
- `internal/discussionparser#requiredDiscussionSections`

**Intent:** Move the decision-record and support-log format out of the Discussion stencil's Step 5 into a deployed spec both writers point at (D2).

Create `contracts/specs/loom-discussion-spec.md`, shaped like `loom-final-summary-spec.md` (an H1, a status blockquote saying it is a pinned contract kept on landing, then sections).
It carries, moved verbatim in substance from `loom-template-discussion.md`'s Step 5 and reworded to name no producer: the decision record's eight H2 sections in order (the eighth, `## Notes for the plan writer`, optional), no frontmatter, and the four compaction rules (`## Decisions` carries Decision and Rationale only with rejected alternatives in the support log; must-cover scenarios go under `## Acceptance criteria`, a scenario naming "a behavior to cover, not a test to write"; a Decision that adds an edge or weakens a guard states "what it can now skip or let through and what bounds it"; no italic prose-coaching); the support log's four H2 sections in order, `## Review rounds` seeded with the single line `_No rounds yet._`.
It also states the two record parts no writer authors: `### Added after Discussion (<by>, <date>): <title>` entries under `## Decisions`, which only `lyx loom decision add` appends and no writer edits or removes, and the `<!-- lyx:carry-over … -->` blocks under `## Open risks`, which only Go writes.
It names `internal/discussionparser` as the parser that checks the format.

Register it in `contracts/specs/specs.go`: an embedded `LoomDiscussionSpec` var and a `{"loom-discussion-spec", &LoomDiscussionSpec}` entry after the two existing ones, so `stencilstore.RelPath` places it at `loom/loom-discussion-spec.md` and the hub pre-run seeds it like the other specs.
`TestRegistry_NamesAndDefaults` gains the name in its want list and `TestRegistry_RelPathPlacesBothUnderLoomFamily` a row for it; `TestRegistry_DefaultsRoundTripTheStamp` iterates the registry and covers it unchanged.

In `loom-template-discussion.md`, Step 5 keeps its heading `## Step 5 — Write the two output files` (twill's directive names "Step 5" as the format step) and its first two lines, then says to write `{{.decision_record_path}}` and `{{.support_log_path}}` in the format `{{.specs_dir}}/loom/loom-discussion-spec.md` defines, that the spec is authoritative and this prompt restates none of it; the two `###` subsections and their lists and rules go.
The header comment lists `specs_dir` as a required marker.
Every other section stays, the fork lens procedure, Step 6's self-check and the gate messages included.

`composePrompt` takes `specsDir` and fills it as the required `specs_dir` marker; `DiscussionSpec` takes `specsDir` and passes it through; `DiscussionTable` takes `specsDir` and sets `Values["specs_dir"]`, since the chair stencil includes the discussion stencil whole.
`loomCLI.wire` passes `websterGeom.SpecsDir` to both closures.
Every test caller passes a specs directory: `TestComposePrompt_RendersMarkers` asserts the rendered prompt names `<specsDir>/loom/loom-discussion-spec.md`, and `TestDiscussionTable` asserts the chair's `Values["specs_dir"]`.

In `wordingClaims`, the `loom-template-discussion.md` row drops its `not a test to write` claim and its appended `attackSurfaceClaims`, whose text moves to the spec, and gains a must claim on `{{.specs_dir}}/loom/loom-discussion-spec.md` scoped to the Step 5 section and a mustNot claim on "`## Auto-mode assumptions`", so the stencil restates none of the format.
A new row for `loom-discussion-spec.md` over `specs.LoomDiscussionSpec` (the claims test imports `contracts/specs`) carries must claims on each required H2 heading, `_No rounds yet._`, `not a test to write`, `Added after Discussion` and `lyx loom decision add`, plus `attackSurfaceClaims`.
The `requiredDiscussionSections` comment cites the spec instead of the stencil's Step 5.
`TestRegistry_RelPathPlacesBothUnderLoomFamily`'s doc says every registered spec rather than both (its name stays, as a rename would only churn), and `internal/loomengine/prompt.go`'s header comment and `composePrompt`'s doc count `specs_dir` among the required markers.
`loom-rubric-discussion-review.md`'s line that sends the reviewer to the writer's stencil to learn what the writer was told not to write also names `{{.specs_dir}}/loom/loom-discussion-spec.md`, where those rules now live; `specs_dir` is already on the rubric marker allowlist.

**ImpactSummary:** the Discussion and seat-table chair prompts now point at a deployed spec through a new required marker; the loomengine re-signs reach only `loomCLI.wire` and the loomengine tests, and the stencil wording claims move with the text.

**Commit:** `1: discussion record format moves into loom-discussion-spec`
