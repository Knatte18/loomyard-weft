# Webster fork prompt: read instruction and pack last

The implementer fork's one-turn read instruction and its kick-start pack now end the fork prompt, since the end of a prompt is what an agent acts on first.
An audit of pack-on forks found a share still reading serially in their first work turn, despite the instruction in the FRESH-READ rule at the top.

## What changes

- With a pack, the prompt's last section is `## Kick-start pack`, which opens with the instruction to read all of it in one turn, in parallel, before any edit, then holds the per-card subsections.
  The instruction carries what the FRESH-READ rule carried: every card file and every listed span, then the target and `Uses:` files no span covers, then the located unresolved glyphs' files.
- The FRESH-READ rule at the top now points at the pack section for what is read before any edit; it keeps the stale-context and Edit-only rules.
- `{{.kickstart_pack}}` moves in `webster-body-implementer.md` from the end of the cards section to the end of the body, after the batch-report section.
  That section still names the report as the fork's last action; only its place in the text moves above the read section.
- Without a pack (pack off, language none), the prompt renders as before.

## Design decisions

- A pack in which no card has a resolved or unresolved entry now counts as no pack: `buildKickstartPack` returns nil and logs `kickstart_pack=empty`.
  Before, such a batch got an all-empty pack section, and with the section now carrying the read instruction it would have ended the prompt with an instruction to read spans that do not exist.
- The pack-on and pack-off fragments are unchanged in how they are selected; only the pack-on FRESH-READ text shrank.

## Docs and tests

- `internal/websterengine/doc.go` describes the pack's new place, the instruction opening the section and the `empty` log value.
- `contracts/stencils/claims_test.go` and the `internal/websterengine` render and begin-batch tests pin the new layout, including the empty-pack case.

## Not run by hand

- A real Webster run with a pack, to confirm the rendered prompt reads as intended end to end.
- The audit's scripts in the hub prime's `.scratch/glyph-audit-2/`, to be re-run on runs after this lands: the share of forks with three or more parallel reads in the first work turn, and exploration calls per card.
