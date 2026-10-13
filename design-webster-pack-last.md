# Webster fork prompt: read instruction and pack last

`webster-pack-last` · Task · enhancement, webster, stencil · running · Publish

## Goal (operator, 2026-10-13)

Move the Webster implementer fork's one-turn read instruction and its kick-start pack to the end of the fork prompt, since the end of a prompt is what an agent acts on first.
This is the cheap next step from the glyph audit after read-all-first (board note `glyph-use-audit`, "Result (audit after read-all-first, 2026-10-13)").

## Seen

- `contracts/stencils/webster/webster-body-implementer.md` inserts the pack mid-prompt (`{{.kickstart_pack}}`, at the end of the cards section), while the one-turn instruction stands in the FRESH-READ rule at the top (`{{.fresh_read}}`, fragments `webster-fragment-fresh-read-pack.md` and `-plain.md`).
- The prompt runs to about 300 lines; the audit found 15 of 29 pack-on forks reading in parallel in their first work turn and 9 still reading serially.

## Change

- With a pack: the prompt's last section is the one-turn instruction ("read all of this in one turn, in parallel, before any edit") followed by the pack; the FRESH-READ rule at the top points to it instead of carrying it.
- Without a pack (pack off, language none, empty pack): the prompt reads as today.
- The batch-report section stays the fork's last action in what it says; only its place in the text moves above the read section.
- Update the tests that pin the stencil's layout and `internal/websterengine/doc.go`'s description of the pack.

## Measure afterwards

The audit's scripts in the hub prime's `.scratch/glyph-audit-2/` re-run unchanged on runs after this lands: the share of forks with three or more parallel reads in the first work turn, and exploration calls per card.
