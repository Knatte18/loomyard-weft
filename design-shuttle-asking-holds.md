# Shuttle: a turn end without output holds the run instead of ending it

`shuttle-asking-holds` · Note · bug, shuttle

## Problem

A turn end without every output file present ends a shuttle run as `asking`, a terminal outcome, while the agent stays alive in its pane.
Nothing waits on that agent any more:
an answer sent into its session makes it work on, unobserved,
and resuming the producer starts a second agent on the same outputs.

Seen 2026-10-06 in `bugfix-sessions` Plan-Write: the plan agent asked a question, ly:orch answered it in the session, the driver resumed the step, and shuttle spawned `plan-2` beside the still-working first agent; ly:orch removed the first one, losing the work done after the answer.

The same rule makes any message to a working agent dangerous: the agent replies, its turn ends without its outputs, and the run halts as `asking`.

## Proposal

A turn end without output never ends a run.

- A turn end with every output file present is done, as today.
- A dead agent and a timeout end the run, as today.
- A turn end without the outputs holds the run with the agent alive: shuttle notifies the parent once that the agent is waiting, and keeps waiting on the same agent.
- An answer to the agent, from the parent, a driver or the operator, is a new turn in the same session; shuttle keeps waiting.
- Resuming the producer attaches to the held run instead of starting a new agent.
- Every message sent to a working agent carries a fixed tail, "Answer briefly, then continue your task", so it replies and works on by itself; senders add it today, and `creel` adds it once it lands.

The cost: a stop no longer halts the run at once; the parent's notice and the run's timeout are the only signals.

## Related

`bugfix-sessions` card 5 removes a live agent of the same outputs before a fresh run, which ends the duplicate but not the lost answer.
The `creel` note owns message delivery and would add the tail.
