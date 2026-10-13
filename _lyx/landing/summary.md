# Webster fork prompt: kick-start pack in its own file, read line last

The implementer fork's kick-start pack used to sit mid-prompt, at the end of the cards section, while its one-turn read instruction stood in the FRESH-READ rule at the top of a prompt of about 300 lines.
The glyph audit found only about half of the pack-on forks reading in parallel in their first work turn.
The end of a prompt is what an agent acts on first, so the pack and its read instruction now come at the end, by reference.

## What changes

- begin-batch writes the pack to its own file, `<batch>-pack.md`, beside the fork prompt file in the webster prompts directory.
  The file opens with the one-turn read instruction ("Read all of this in one turn, in parallel, before any edit"), the spans, whole-file and unresolved-glyph reads that instruction covers, then one subsection per card as before.
  The pack file is written before the prompt, so the prompt never names a missing file.
- The fork prompt's very last line, under a `## Start here` heading after the batch-report section, is "Now read `<pack file>` and do what it says."
  The `kickstart_pack` marker moved from the end of the cards section to the end of `webster-body-implementer.md`, and is filled by the new stencil `webster-fragment-pack-read`.
  The batch-report section still names the report as the fork's last action.
- The pack-on FRESH-READ rule points at the pack file instead of carrying the read instruction, and the pack locate step refers to "the kick-start pack file".
- `RenderForkPrompt` takes the pack path and returns the pack file content beside the prompt; a nil pack returns no pack file.
- A batch where no card has a resolved or unresolved entry now counts as no pack: the prompt renders as with the switch off, no pack file is written, and the begin-batch trace line logs `kickstart_pack=empty`.
  Before, such a batch got a pack section saying each card named no member glyph.

Without a pack (switch off, language `none`, empty pack) the prompt's bytes are unchanged, and the recovery prompt never takes a pack.

## Docs and tests

`internal/websterengine/doc.go`'s kick-start pack section describes the pack file, the read line and the `empty` log state.
The stencil claims and marker tests pin the new layout: the read line ends the body, FRESH-READ no longer carries the read instruction, and the pack file opens with it.
The render tests check that a prompt with a pack ends with the read line and that a nil pack renders no pack file.
`TestBeginBatch_KickstartPack` reads both files, checks that no pack file is written when there is no pack, and covers the empty-pack case.

## Measure afterwards

The glyph audit's scripts can re-run unchanged on runs after this lands, comparing the share of forks with three or more parallel reads in the first work turn and the exploration calls per card.
