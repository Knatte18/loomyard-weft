# Kick-start pack per card, read when the card starts

`kickstart-per-card` · Task · enhancement, webster, stencil · running · Webster

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

- Today's pack is one file per batch, `<batch>-pack.md`, already sectioned under each card's path; only the instruction makes it batch-wide.
  The cheapest form is therefore a new fragment text behind the switch: "before card N's first edit, read card N's section of the pack in one parallel turn", with no change to the pack file.
  The heavier form splits the pack into one file per card and hands each to the fork when it starts that card.
- Replace the read-all-first instruction with a per-card one: before the card's first edit, read its whole pack in one parallel turn.
- Frame the card pack as complete for a start, so the fork reads further only when the card needs it.
- Putting the span code itself into the pack was considered and rejected; see the operator decision on variant 2 below.

## Trade-off raised by the operator (2026-10-15): all at once lets the fork plan the whole batch

For reading everything first: the fork sees how the cards interact, plans the batch in one go, and fetches in one parallel turn instead of many small ones.
Against it: forks did not do it in the audit; code read in turn 2 is carried and paid as cache reads in every later turn, also through cards that do not need it; and line numbers go stale after the first edit anyway.
Two variants to measure against today's layout behind the switch:

1. Overview first, code per card (the orch's recommendation): at batch start the fork gets every card's intent and glyph list, no code, to plan the batch; it reads a card's spans when it starts that card.
   Those spans must be resolved fresh at card start, since an earlier card's edits shift line numbers: Claude Code's Edit matches by text and is unaffected, but Read takes line offsets that do not follow edits.
   So the card's pack comes from one call at card start (`lyx quarry resolve` over the card's glyphs, or a new `lyx webster` verb that renders the card's pack against the current tree), and the fork reads its spans in one parallel turn.
2. All at once as today, but with the span code inside the pack file, so it is one read instead of about 37 and gets done; the context is as heavy as today's intended read.
- Keep the layout behind the `webster.yaml` switch, as `kickstart_pack_file` is today, so it can be measured against the current layout with the audit scripts.
- History: `113593990` had a per-card pack inline without a read rule (arm `mid` in the 2026-10-13 audit, 9 forks); this differs in giving each card its pack when that card starts.

## Operator decision (2026-10-15): scope

Build variant 1 above, overview first and code per card, with each card's spans resolved fresh at card start and read in one parallel turn.
Put it behind a `webster.yaml` switch beside `kickstart_pack_file`, so it can be measured against today's layout; the begin-batch `webster: kickstart pack` trace line records which layout a batch used.
Variant 2 (span code inside the pack file) is not part of this task.

## Operator decision (2026-10-15): no code in the pack

Variant 2 is rejected, not deferred: code pasted into a pack goes stale fast.
It would also save no reads: Claude Code refuses an Edit on a file the agent has not read itself in the session, so a fork must Read every file it edits whatever the pack holds.
The pack points at code and never stands in for reading it.
