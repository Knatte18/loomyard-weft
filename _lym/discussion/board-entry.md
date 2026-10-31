# Board entry `hold-notice-operator-turn`: No hold notice for a turn the operator started in the pane

The shuttle hold notice is held back when the operator typed the turn's prompt

## Seen (operator, 2026-10-31)

The operator talked to the writer of `tapestry-rename` in its pane, and the writer answered and ended its turn without its output files.
lym then sent the orch a shuttle hold notice ("ended a turn without its output files and is held"), though the turn was the operator's own conversation, not a stall.
The notice does not say who the agent talked to, so the orch could not tell and had to ask the operator.

## Idea

A turn that began with the operator's own input in the pane sends no hold notice: lym tells an operator-typed prompt from the injects and mail it sends itself, and holds back the notice for that turn.
The notice still goes out when a later turn, started by lym or by mail, ends without the output files.
