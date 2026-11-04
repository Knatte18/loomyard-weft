# Discussion writers start their exploration with explore subagents

`discussion-explore-agents-first` · Task · enhancement, stencil, loom · running · Drift-Recheck

## Seen (`docs/research/writer-lookups-measure-2026-11-04/report.md`)

- The Plan writers follow the stencil's lookup guidance: 3 of 3 sent one early batch of 3 or 4 `sonnet` lookup subagents.
- The Discussion writers mostly do not: 2 of 3 full sessions started no lookup subagent and explored in their own context, in chains of 15 and 20 serial turns.
  One Discussion session took in 170k tokens of lookup results and carried them through every later turn.
- Carry, the cost of keeping what lookups returned in context for the rest of the session, was $11.25 of the writers' $23.76, larger than the lookups' own cost.
  An early explore subagent returns only what is relevant, so its main saving is a smaller context for the whole session, not cheaper lookups.

## Idea (operator and orch, 2026-11-04)

The Discussion-Write stencil asks the writer to start its exploration with explore subagents: one batch of independent questions to fresh `subagent_model` subagents up front, before it reads code in its own context, as the Plan writers already do.
