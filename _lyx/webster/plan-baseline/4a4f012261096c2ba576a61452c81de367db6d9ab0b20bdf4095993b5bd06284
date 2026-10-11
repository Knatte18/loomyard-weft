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

`loom-template-discussion.md` gains `{{.twill_directive}}` on the line that is today the blank line between Step 4's `{{.mode_rules}}` line and the `## Step 5` heading, and `{{.twill_write_scope}}` on the line that is today the blank line between the `{{.support_log_path}}` bullet under `## What you may write` and the `Two narrow carve-outs` paragraph.
Each marker replaces that blank line rather than adding a line, so an empty marker renders the blank line it replaced and the `single` and chair prompts keep their bytes.
The section's own sentences stay as they are.
The banner comment lists both as optional markers rendered by `internal/twilldirective` through `internal/loomengine`, beside the three existing optional ones.

`composePrompt` takes `twillDirective` and `twillWriteScope`, puts them in the value map under `twilldirective.MarkerName` and `twilldirective.WriteScopeMarkerName`, and adds both names to the `FillOptional` optional list, so an empty value renders as nothing.
A non-empty value goes into the map with one newline before and one after it, so the block sits in a paragraph of its own, set off by one blank line on each side, and the write-scope paragraph never continues the bullet above it.
`DiscussionSpec` passes two empty strings for now; card 9 fills them for `twill`.
`DiscussionTable` adds both marker names to the table's `Optional` list with no value, since the chair includes the discussion template whole and `FillWith` checks an included block's markers too.

`TestComposePrompt_RendersMarkers` passes the two new arguments.
Its existing rows pass them empty and assert the rendered prompt carries neither marker name and no twill text, and pin the bytes at both insertion points: the mode-rules text is followed by exactly one blank line and then `## Step 5`, and the support-log bullet by exactly one blank line and then `Two narrow carve-outs`.
One added case passes a sentinel directive and a sentinel scope and asserts the directive lands after the mode rules and before `## Step 5`, and the scope inside `## What you may write` after the support-log bullet and before the carve-outs, each with exactly one blank line on either side.
`TestCompose_FrictionDirective` passes empty strings at its call.
`TestDiscussionTable` asserts, in both modes, that the chair prompt carries no `twill` text, beside its existing no-`{{` assertion, and that the support-log bullet is followed by exactly one blank line and then `Two narrow carve-outs`, and `## Step 5` is preceded by exactly one blank line.

`wordingClaims`' discussion-template row gains `{{.twill_directive}}` scoped to the `## Step 4 — How to get answers` section and `{{.twill_write_scope}}` scoped to the `## What you may write` section.

**ImpactSummary:** `composePrompt` has three callers, all in `internal/loomengine` and all on this card; the rendered `single` and chair prompts are unchanged.
