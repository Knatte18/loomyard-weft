# Plan and Discussion writers explore through parallel Sonnet agents that return only what is relevant

`planner-fork-lookups` · Note · research, loom

## Seen (orch, 2026-11-01)

In a sample of 28 sessions (`.scratch/batching-breach.md` of the batching study), the single largest cost was the Plan writer of `run-verbs-shared` in `plan-split-bench`: 31 chained grep turns at 390k to 530k context and 15 one-file-per-turn fix turns at 630k to 668k.
Every lookup the writer makes is paid at its whole context, so exploration late in a large session is the expensive part of planning.
Discussion-Write explores the code more than any other role, so the same cost applies there.

## Design (operator, 2026-11-01)

- Plan-Write and Discussion-Write start several fresh, cold-start Sonnet explore subagents in parallel, in one message, each with its own questions.
- Each explore agent returns only what is relevant: a list of the files, spans and facts the writer needs, with paths and line ranges, not a dump of what it read.
- The writer reads only what those lists name, in one message, and writes the plan or discussion with the whole overview in its own session.
- The writer is encouraged to work this way, not commanded: the stencils say when it pays, such as broad exploration up front or a batch of open questions late in a large session.
  The writer keeps its own lookups and uses them when it judges a lookup small or when it needs to see the code itself.
- Forks do not serve here: a fork inherits the writer's whole context and saves nothing.

## Research

- From the Plan-Write and Discussion-Write transcripts of landed tapestry runs, count the writers' exploration turns, their context and cost, and how many could have gone to parallel explore agents up front.
- Bench on one or two replays (the `plan-split-bench` inputs) for both roles: the writer allowed subagents but given no stencil guidance, against the writer with the guidance above.
  Compare whether the guidance changes what the writer does, and cost, turns, peak context and review findings.
- Name the risk: an explore agent's list may leave out what the writer would have seen in the code, so count review findings that trace to a missed fact.

## What the bench writer did (orch, 2026-11-01)

The `psb-rv-plan` writer ran on `opus[medium]`; its Agent calls were one fork writing the plan and the `twill-plan` lens reviews, none for lookups.
Plan-Write ran in fork mode, where lym's hook (`steerAgentNonForkDeny` in `claudeengine/settings.go`) allowed only fork subagents, so a fresh explore agent was refused.

The bench runs after task `darn-sonnet-helpers` lands, which removes that hook for every role.
