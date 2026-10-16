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
