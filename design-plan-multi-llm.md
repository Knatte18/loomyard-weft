# Plan-Write on MultiLLMProducer: advisors gather facts and check cards

`plan-multi-llm` · Note · feature, loom, stencil

## Seen

Plan gate rejects and Plan-Review rounds come mostly from facts the planner got wrong or missed: an uncovered caller, a verify line in the wrong module, a nested fixture module, a card over its test budget.
Plan is one document that must hang together, so two agents writing it at once would conflict.

## Idea

Plan-Write runs on `MultiLLMProducer` (task `multi-llm-producer`), the planner as the `chair`, with advisors doing work the chair would otherwise do itself:

1. A code-reading advisor maps, for each package the plan touches, its callers, tests, nested modules and verify commands, and answers the chair's questions from that map.
2. A card-checking advisor reads each card as the chair finishes it and checks its scope, `Edit:` list, verify lines and test budget, so the chair fixes it while its context is fresh rather than in a later review round.

A third shape, advisors writing the cards of their own batch from the chair's overview, is faster on large tasks but makes the cards hard to keep consistent; left out unless the first two fall short.

## When

Operator, 2026-10-08: later, after `discussion-multi-llm`.
Plan runs on `fable[high]` as a single agent from 2026-10-08; this is the next step if that does not reduce plan gate rejects and Plan-Review rounds.
