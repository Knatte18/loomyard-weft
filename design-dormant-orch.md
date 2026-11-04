# The orch lies dormant and wakes only when a run needs help

`dormant-orch` · Note · feature, orch, dobby, arbiter

## Seen (operator, 2026-10-31)

The orch is a long-lived, heavy session that handles board work, run notices and escalations alike.
Board work moves to a short-lived board agent: the prime's `board` button opens one with the skills `ly:board-policy` and `ly:board` (task `board-agent-button`, `2def9684`, landed 2026-11-04), and dobby already does the mechanical run work.

## Idea (long term)

Dobby takes everything mechanical, and a capable agent spawned per case, as the arbiter is for a halted stop, settles the doubtful ones.
One orch per beam, the prime included, is kept, since it is how the operator runs the hub from the phone.
It is not started on demand: it lives as today but lies dormant, and is woken only when a run needs help or the operator talks to it (operator, 2026-10-31).
