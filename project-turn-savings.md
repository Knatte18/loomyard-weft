# P1 Fewer agent turns, fewer tokens

Rendered 2026-11-01 18:17 UTC; every age below is as of then.
Under: [Board](README.md).

`turn-savings` · Age 0m · Owner `neve`

Token savings by cutting agent turns, above all late turns at large context

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (0)

Concrete and claimable; only a task can run.

### Ready (0)

Waits on nothing open; can start now.

_None._

## Notes (1)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Enhancements (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E7 |  | **A hook nudges an agent back to batching the moment it breaks the rule**<br>• Above about 150k context agents drift into grep-then-slice chains and one-fix-per-turn edits, about a third of context tokens in a 28-session sample; a PostToolUse hook adds one line on a breach, with sharper rule wording | [`batching-breach-nudge`](design-batching-breach-nudge.md) | 8m<br>0m | claude, shuttle |
