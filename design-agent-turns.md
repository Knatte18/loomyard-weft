# Fewer agent turns, Webster forks first

`agent-turns` · Project · enhancement, webster

## Goal (operator, 2026-11-01)

Agents, Webster's implementer forks first, finish their work in fewer turns.
A session's cost grows with its turns times its context, and a Webster fork's context runs at 75k to 475k tokens a turn, so every turn cut is worth much.
The members are taken one at a time, each measured against the numbers below before the next.

## Seen (orch, 2026-11-01)

`tools/tokencount` on the three latest full tapestry runs (`warp-visibility`, `interactive-recipe`, `run-verbs-shared`): Webster forks cost $85 of $308, the largest part, almost all of it cache reads.
Fork starts are 74k to 119k (Merriam's base, inherited cheaply), peaks 125k to 476k, and 20 to 164 messages a fork; the growth inside a fork is the cost, not its base.
`tools/forkaudit` on the same runs' 27 forks: 31 turns, 23 partial reads, 18 greps and 6 whole-file reads per card; about 1.7 read, grep and edit calls per turn, against 1.37 before `turn-batching`.

A turn-by-turn reading of the five heaviest forks (165, 116, 103, 90 and 84 turns) found where the turns went:
- Edits made with `sed -i`, awk, perl or scratch-file splices, about one a turn; 66 to 90% of the edits in the three heaviest forks, against 2.3 edits a turn in the two forks that used the Edit tool.
- A sweep for a retired key that skipped test files, so stale fixtures in about ten packages surfaced one gate failure at a time: 69 gate-loop turns in one fork.
- Gate output too large to read in one result, about 10 turns spent only reading it, and 14 lint-only turns.
- One file read in three to five slices over as many turns; about 80 of 208 single-call read turns could have gone a turn earlier.

Measures already rejected: shorter forks (the 300k batch trial showed fewer, larger forks cost less), quarry lookups (agents used them on top of their own searching), and cold-start batches (Weaver), which would pay full price for what forks inherit from Merriam.
