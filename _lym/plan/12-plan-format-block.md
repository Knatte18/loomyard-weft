# Card 12 — plan-format-block

**Create:**
- `contracts/stencils/loom/loom-block-plan-glyphs.md`
- `contracts/stencils/loom/loom-block-plan-format.md`
- `contracts/stencils/loom/loom-block-plan-overview.md`
- `contracts/stencils/loom/loom-block-plan-cards.md`
- `contracts/stencils#LoomBlockPlanGlyphs`
- `contracts/stencils#LoomBlockPlanFormat`
- `contracts/stencils#LoomBlockPlanOverview`
- `contracts/stencils#LoomBlockPlanCards`

**Edit:**
- `contracts/stencils/loom/loom-template-plan.md`
- `contracts/stencils/loom/loom-rubric-plan-review.md`
- `contracts/specs/loom-plan-spec.md`
- `contracts/specs/webster-spec.md`
- `docs/overview.md`
- `contracts/stencils#TestStencils_TopLevelMarkers`
- `contracts/stencils#TestHubGenericStencils_NameNoGoTiers`

**Extends:**
- `contracts/stencils#entries`

**Intent:** Move the card format and the glyph lookup rule out of the Plan-Write stencil into included blocks (D12), so `Plan-Write`, the part writer and the parts chair read one copy (PATTERN-stencil-ownership).
`Plan-Write`'s composed prompt stays the same line for line, in today's order (the parent's added decision on Plan-Write's prompt): the moved text is byte-identical, no cross-reference is rewritten, and each block is included where its text stands today.

**The blocks.** The seat engine includes a block whole at one position and one level deep (`internal/seatengine#validateStencil`), and three lines inside the moved text do not fit every including stencil, one naming `Plan-Write`'s Step 3 and two binding `Plan-Write` and the chair but not a part writer, so the text splits into four blocks at those lines, each moved verbatim out of `loom-template-plan.md`:
1. `loom-block-plan-glyphs`: the `### Look up glyphs with `lym quarry` — never spell one from memory` subsection of Step 2 from its heading through "**You never spell a glyph — you copy a line verbatim out of a quarry answer.**" (today's lines 54–68), with no marker;
2. `loom-block-plan-format`: from `### What a card is` through the `### On-disk layout` line "cards run `1..M` with no gaps." (today's lines 78–110), with the marker `{{.specs_dir}}`;
3. `loom-block-plan-overview`: from `### `00-overview.md`` through "Body: a short task-framing paragraph, …" (today's lines 113–165), with the marker `{{.specs_dir}}`;
4. `loom-block-plan-cards`: from "A symbol `Rename` needs the hub's `rename_tools` in `language.yaml`; …" through the three lines after the `### Minimal skeleton` skeleton (today's lines 167–279), with the markers `{{.specs_dir}}` and `{{.gate_tiers}}`.

The three lines left in the template are the glyph rule's "This is the hard rule behind the glyph spelling rules Step 3 spells out below: …", which names a step, the `### On-disk layout` sentence "Insert or remove a card only with `lym loom insert-card` or `lym loom remove-card`, …", and the `### 00-overview.md` sentence "When the decision record has a `## PATTERN changes` section, copy its bullets into the overview's `## PATTERN changes` section, …".
Each including stencil states its own rule for them.
No block names a `Step`.

Each block file opens with an HTML comment saying which plan-format block it is, that `loom-template-plan`, `loom-template-part-writer` and `loom-template-parts-chair` pull it in with the bare `{{template "<name>"}}`, that the fill hands it the including seat's value map, which markers it declares, and that it names no step of the stencil around it.
The fill strips that leading comment, and the file ends without a trailing newline, as `contracts/stencils/seat/seat-directive-chair.md` does, so an include on its own line gives back exactly the lines it replaced.

**The template.** In `loom-template-plan.md`, `{{template "loom-block-plan-glyphs"}}` replaces today's lines 54–68, followed by the kept hard-rule line, the line "The verbs above are the whole of what `lym quarry` offers you …" and `{{.twill_directive}}`, as today.
Step 3 keeps its heading, the blank line after it, its three opening lines and the blank line after them, then holds `{{template "loom-block-plan-format"}}`, the kept insert-card sentence, a blank line, `{{template "loom-block-plan-overview"}}`, the kept `PATTERN changes` sentence and `{{template "loom-block-plan-cards"}}`, each on its own line; Step 4 follows unchanged.
The header comment adds that the stencil includes the four plan-format blocks, and that `specs_dir` and `gate_tiers` reach them through the same value map.
Each block embeds beside `LoomTemplatePlan` under its own variable above, with an entry in `entries`.

**Pointers to the format.** Each place that says the LLM-facing plan format is pinned in `contracts/stencils/loom/loom-template-plan.md` is reworded to say it is pinned in the plan-format blocks `loom-block-plan-format`, `loom-block-plan-overview` and `loom-block-plan-cards`, which `loom-template-plan.md` includes: `contracts/specs/loom-plan-spec.md`'s status line, its "This format is produced by `Plan-Write`" line and its reference-list entry for the stencil; `contracts/specs/webster-spec.md`'s opening pointer and its reference-list entry; and `docs/overview.md`'s webster entry.
In `contracts/stencils/loom/loom-rubric-plan-review.md`, only the pointer "The plan writer's own stencil is `{{.stencils_dir}}/loom/loom-template-plan.md`." changes, so it also names the three blocks under `{{.stencils_dir}}/loom/` that the stencil includes, with no other rubric change.

**Tests.** `TestStencils_TopLevelMarkers` gains `{"loom-block-plan-glyphs", LoomBlockPlanGlyphs, nil}`, `{"loom-block-plan-format", LoomBlockPlanFormat, []string{"specs_dir"}}`, `{"loom-block-plan-overview", LoomBlockPlanOverview, []string{"specs_dir"}}` and `{"loom-block-plan-cards", LoomBlockPlanCards, []string{"gate_tiers", "specs_dir"}}`.
`TestHubGenericStencils_NameNoGoTiers` adds the four blocks to the stencils it scans, since the moved text, the gate-tier text among it, now lives there.
While it moves the text, the card checks once that the move kept the prompt: it composes the lead's prompt of `loomengine.PlanTable` with `seatengine.Prompts` over the same table values before and after the move and compares the two line for line; nothing is committed for it.

**ImpactSummary:** Plan-Write's composed prompt is unchanged line for line; four blocks are new and registered, and the pointers to the format name them.
