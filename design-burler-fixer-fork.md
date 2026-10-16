# Fork the Burler fixer from the reviewer session

`burler-fixer-fork` · Note · enhancement, burler

## Seen

Carried over from the removed note `model-downgrade` (2026-10-15).
Each Burler fixer is a fresh session that reads the whole context the reviewer just read, part of the $1.9 per run in the Burler cost breakdown.
Skipping the fixer on an empty review is already built (`internal/burlerengine/fix.go`).

## Idea

Fork the fixer from the reviewer's session, so it inherits the reviewer's cache prefix instead of reading the context again.
A fork inherits the reviewer's model, so this rules out a separate fixer model for a forked fixer.

## Other open items from `model-downgrade`

- Plan and Discussion fixers to `sonnet[medium]`: estimated about $4 over six runs, not benched.
- Fan on or off per round (fan in round 1, solo after) does not exist and needs code.
- A per-batch Webster model (a cheap model only for mechanical cards) needs code in `internal/websterengine`.

## Operator view, not a decision (2026-10-16)

The operator wants a far more critical fixer that uses the Twill setup to review its own work before the next review round.
`perch-review-analysis` (issue #676) proposes the same: after its fix the fixer spawns forks of itself that check consistency, claims and the way forward, aimed at late findings an earlier fix caused.
This pulls against this note's cost idea, since a Twill fixer spends more tokens per round to need fewer rounds.

## Operator proposal, not a decision (2026-10-16): reviewer master that forks

Taken from an agent's suggestion in a discussion on another machine.
The Burler reviewer explores the code as today, and just before it reasons out its review it forks into N review forks, one per lens.
N is 1 for now, since no fan-out is on: one general lens, the review we have today, but lenses must be supported.
The master waits; when the forks are done it gets their reviews and fixes the findings critically.
Then the master forks again, and the forks give a Twill-like critical review of the fix itself, much like the Twill self-review the Discussion and Plan writers do today.
The fix itself may also be done by a fork, so the master does not hold the reasoning behind the fix either.
The gain the other agent pointed out: the reviewer master already has every file in context, so no fresh fixer reads it all again, which saves tokens and time.
A smaller first step: keep today's reviewer and fixer split, and make only the self-review of the fix part of the fixer agent.
The operator asked for an audit of whether this is feasible and worth it (2026-10-16).
