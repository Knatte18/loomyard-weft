# Darn measurement rule covers a run with no parent and bars subagents from measurement calls

`darn-measure-rule-gaps` · Task · bug, darn, stencil · running · Friction-Reflect

## Seen

The landing that has the darn writer ask before an entry's measurement gate calls (`ef9062d91`) closed Code-Review over two accepted-circling findings on the new stencil rule:

- `measurement-rule-no-parent-path-drops-deliverable`: with no parent session, the measurements the entry asks for are never produced.
- `ungated-subagents-bypass-measurement-rule`: the gated fragment has no sentence of its own barring subagents from measurement calls.

## Fix

- Give the rule a path for a run with no parent, so the entry's measurements are still produced or the run stops with a way forward.
- Add a sentence to the gated fragment that bars subagents from measurement calls.
