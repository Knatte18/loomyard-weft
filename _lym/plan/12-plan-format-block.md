# Card 12 — plan-format-block

**Create:**
- `contracts/stencils/loom/loom-block-plan-format.md`
- `internal/loomengine/testdata/plan-prompt.golden`
- `plan:contracts/stencils#LoomBlockPlanFormat` -> `var LoomBlockPlanFormat []byte`

**Edit:**
- `contracts/stencils/loom/loom-template-plan.md`
- `contracts/stencils#TestStencils_TopLevelMarkers`
- `contracts/stencils#TestHubGenericStencils_NameNoGoTiers`
- `internal/loomengine#TestPlanTable`

**Extends:**
- `contracts/stencils#entries`

**Intent:** Move the card format and the glyph lookup rule out of the Plan-Write stencil into one included block, `loom-block-plan-format` (D12), so `Plan-Write`, the part writer and the parts chair read one copy (PATTERN-stencil-ownership).
`Plan-Write`'s composed prompt keeps its content: the moved text is byte-identical except the cross-references the move rewrites, named below.

**The block.** `contracts/stencils/loom/loom-block-plan-format.md` opens with an HTML comment saying it is the plan-format block that `loom-template-plan`, `loom-template-part-writer` and `loom-template-parts-chair` pull in with the bare `{{template "loom-block-plan-format"}}`, that the fill hands it the including seat's value map, that it declares the two required markers `{{.specs_dir}}` and `{{.gate_tiers}}`, and that it names no step of the stencil around it.
Its body, in this order, is moved verbatim out of `loom-template-plan.md`:
1. the `### Look up glyphs with `lym quarry` — never spell one from memory` subsection (today's lines 54-70 of Step 2), with one rewrite: "This is the hard rule behind the glyph spelling rules Step 3 spells out below:" becomes "This is the hard rule behind the glyph spelling rules below:";
2. every subsection from `### What a card is` through `### Minimal skeleton` and the three lines after its skeleton (today's lines 78-279), with two sentences left out and kept in the template, as below: the `### On-disk layout` sentence "Insert or remove a card only with `lym loom insert-card` or `lym loom remove-card`, …, never by renaming, rewriting under a new name or deleting a card file by hand.", and the `### 00-overview.md` sentence "When the decision record has a `## PATTERN changes` section, copy its bullets into the overview's `## PATTERN changes` section, one per entry, without the added-after-Discussion suffix; the plan gate refuses a mismatch."
Those two sentences are the only moved text that binds `Plan-Write` and the chair but not a part writer, which is why each including stencil states its own rule for them.
The block names no `Step`.

**The template.** In `loom-template-plan.md`, Step 2 ends at "Issue every search and read you can name in one message; wait for a result only to choose the next call.", followed by `{{.twill_directive}}` on its own line as today.
Step 3 keeps its heading and its three opening lines, then holds `{{template "loom-block-plan-format"}}`, then the two kept sentences verbatim as one paragraph with no heading of its own; Step 4 follows unchanged.
The template engine includes a block whole at one position and one level deep (`internal/seatengine#validateStencil`), so the glyph subsection, which sat in Step 2, now opens the card-format text in Step 3: the composed prompt keeps every line, and only the order of that subsection and the place of the two kept sentences change.
The header comment adds that the stencil includes the plan-format block, and that `specs_dir` and `gate_tiers` reach it through that block.
`LoomBlockPlanFormat` embeds the block beside `LoomTemplatePlan`, with an entry in `entries`.

**Tests.** `TestStencils_TopLevelMarkers` gains `{"loom-block-plan-format", LoomBlockPlanFormat, []string{"gate_tiers", "specs_dir"}}`.
`TestHubGenericStencils_NameNoGoTiers` adds `loom-block-plan-format` to the stencils it scans, since the gate-tier text moved there.
`TestPlanTable` gains an assertion on the lead's composed prompt from `seatengine.Prompts` over its fixed table values: its lines, as a multiset, equal those of `internal/loomengine/testdata/plan-prompt.golden`, the composed prompt for the same values captured from the stencil before the move, except the one rewritten cross-reference line; so a dropped, added or altered line in any moved section fails it.
The card writes that golden file first, from the unmoved stencil, then moves the text.
It also asserts that the block's registered default holds no `Step ` and no `{{.` marker but `specs_dir` and `gate_tiers`, the rule that the block names no step of the stencil around it.

**ImpactSummary:** Plan-Write's prompt keeps every line, the glyph subsection now heading Step 3's card format; the block is new and registered.
