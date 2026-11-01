# Trial: agents settle every edit before the first, then gate once

`plan-edits-before-gate` · Task · enhancement, heddle, stencil, claude · draft

## Seen (orch, 2026-11-01)

The five heaviest Webster forks of the three latest tapestry runs worked in small steps: a few edits, a gate run, the next failure read, a few edits more, though each fork's card already names what to change.
That is why edits per edit turn stayed near 2.3 and gate loops ran long; `edit-with-edit-tool` gathers one file's edits into one message, and the batching sentence gathers calls whose arguments are known, but nothing asks for all of a change's edits to be settled before the first one.
A Reddit report the operator read cut agents' turns by having them plan every change before making any.

## Idea (operator and orch, 2026-11-01)

- An imperative instruction, delivered as a heddle entry in the rendered `CLAUDE.local.md`: read what the change needs, decide every edit across every file the card or task touches, make them all, then run the gate once and fix everything its report shows in one round.
- It is a heddle text with no code change, so it runs as a trial in wave 3, measured alone: gate runs per card and edit-and-gate rounds, on Webster forks with `tools/forkaudit` and on darn writers with the scripts of `docs/research/turn-audit-2026-10-28`.
- Should the trial show the effect, the instruction moves into the stencils lym ships, since it holds for every hub.
