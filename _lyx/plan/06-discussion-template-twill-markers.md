# Card 6 — discussion-template-twill-markers

**Edit:**
- `contracts/stencils/loom/loom-template-discussion.md`
- `internal/loomengine#composePrompt` -> `func composePrompt(stencilsDir, slug, decisionRecordPath, supportLogPath, patternDirective, frictionDirective, parentDirective, twillDirective, twillWriteScope string, autonomous bool) ([]byte, error)`
- `internal/loomengine#DiscussionSpec`
- `internal/loomengine#DiscussionTable`
- `internal/loomengine#TestComposePrompt_RendersMarkers`
- `internal/loomengine#TestCompose_FrictionDirective`
- `internal/loomengine#TestDiscussionTable`
- `contracts/stencils#wordingClaims`

**Uses:**
- `plan:internal/twilldirective#MarkerName`
- `plan:internal/twilldirective#WriteScopeMarkerName`

**Intent:**
Give the discussion template the two optional twill markers of decision record D2 and thread them through the composer, rendering nothing for `single` and for the chair so their prompts stay byte-identical.

`loom-template-discussion.md` gains `{{.twill_directive}}` on its own line between the end of Step 4's `{{.mode_rules}}` paragraph and the `## Step 5` heading, and `{{.twill_write_scope}}` on its own line directly below the two-file list under `## What you may write`, before the carve-outs paragraph.
The section's own sentences stay as they are.
The banner comment lists both as optional markers rendered by `internal/twilldirective` through `internal/loomengine`, beside the three existing optional ones.

`composePrompt` takes `twillDirective` and `twillWriteScope`, puts them in the value map under `twilldirective.MarkerName` and `twilldirective.WriteScopeMarkerName`, and adds both names to the `FillOptional` optional list, so an empty value renders as nothing.
`DiscussionSpec` passes two empty strings for now; card 8 fills them for `twill`.
`DiscussionTable` adds both marker names to the table's `Optional` list with no value, since the chair includes the discussion template whole and `FillWith` checks an included block's markers too.

`TestComposePrompt_RendersMarkers` passes the two new arguments: empty in its existing rows, asserting the rendered prompt carries neither marker name and no twill text; and one added case passing a sentinel directive and a sentinel scope, asserting the directive lands after the mode rules and before `## Step 5`, and the scope lands inside `## What you may write` after the support-log bullet and before the carve-outs.
`TestCompose_FrictionDirective` passes empty strings at its call.
`TestDiscussionTable` asserts the chair prompt carries no `twill` text, beside its existing no-`{{` assertion, in both modes.

`wordingClaims`' discussion-template row gains `{{.twill_directive}}` scoped to the `## Step 4 — How to get answers` section and `{{.twill_write_scope}}` scoped to the `## What you may write` section.

**ImpactSummary:** `composePrompt` has three callers, all in `internal/loomengine` and all on this card; the rendered `single` and chair prompts are unchanged.
