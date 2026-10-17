# Creel replies always go back by post

`creel-reply-discipline` · Note · enhancement, creel, stencil

## Seen (orch, 2026-10-17)

In a demo, `ly:orch` posted #3 to a fresh Claude strand `ly:demo`, whose body asked for a reply by `lyx creel post --reply`; the agent read it and replied as #4.
It replied by post only because the body told it how.
An inject (#5) asking for `PONG` was answered in the receiver's own pane, to the operator, and never reached the sender: an inject is typed as a user line and has no reply path.
`lyx creel read` prints the message with its id but says nothing about answering by `post --reply`.
Creel already limits inject: only EXTERNAL callers, programs and orch strands may inject into another strand (`lyx creel inject --help`).

## Operator view, not a decision (2026-10-17)

Creel needs a strict regime, so that an agent that sends something to another agent never ends up without an answer because the receiver wrote its answer in its own pane, to the operator.

## Idea (orch recommendation)

- `lyx creel read` prints, with each message, the command that answers it (`lyx creel post --reply <id> <text>`), and says when the sender waits on the reply (`--wait-reply`).
- The agent stencils state the rule once, in a shared block: a creel message is answered by `post --reply`, never in the pane; a line in the pane is for the operator.
- An inject never carries a question that expects an answer from the receiver; a sender that wants an answer posts.
- A message whose sender waits and that goes unanswered past a bound is journaled, so the orch can see it.

## Open

- Whether this belongs in `creel-notices` (stage 2), where the orch answers the watcher's note request by post, or in `creel-agents` (stage 3), where agents move onto creel.
