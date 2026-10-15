# Kick-start pack per card, read when the card starts

`kickstart-per-card` · Note · enhancement, webster, stencil

## Seen

The 2026-10-15 pack audit (`.scratch/glyph-audit-3/`, to be committed under `docs/research/`) measured 24 Webster forks on the pack-file layout (`1e39afc10`).
Every fork opened the pack file in turn 2, but the median fork reached only about a quarter of its listed spans before its first edit, and only 7 of 24 read in parallel in their first work turn (52% under the earlier read-all-first layout).
A batch pack holds on average about 37 spans per fork, so "read every card and every span first" (`793c6a8ee`, today's `webster-fragment-locate-pack.md`) asks a fork to front-load a whole batch's code.
The pack entries were correct at batch start: 801 of 823 checked spans exact, 22 declaration-only, none stale or wrong.

## Operator view, not a decision (2026-10-15)

Give the pack per card instead of per batch.
The batch gets one general instruction.
Each card then says: read this card's kick-start pack first; it is, to begin with, everything you need for this card.

## Idea

- Split the pack by card and hand each card's part to the fork when it starts that card, not all at batch start.
- Replace the read-all-first instruction with a per-card one: before the card's first edit, read its whole pack in one parallel turn.
- Frame the card pack as complete for a start, so the fork reads further only when the card needs it.
- Combine with putting the span code itself into the pack (separate idea from the same audit), so a card's pack is one read.
- Keep the layout behind the `webster.yaml` switch, as `kickstart_pack_file` is today, so it can be measured against the current layout with the audit scripts.
- History: `113593990` had a per-card pack inline without a read rule (arm `mid` in the 2026-10-13 audit, 9 forks); this differs in giving each card its pack when that card starts.
