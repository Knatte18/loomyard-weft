# Plan and Discussion writers explore through parallel Sonnet agents that return only what is relevant

`planner-fork-lookups` · Note · research, loom

## Seen (orch, 2026-11-01)

In a sample of 28 sessions (`.scratch/batching-breach.md` of the batching study), the single largest cost was the Plan writer of `run-verbs-shared` in `plan-split-bench`: 31 chained grep turns at 390k to 530k context and 15 one-file-per-turn fix turns at 630k to 668k.
Every lookup the writer makes is paid at its whole context, so exploration late in a large session is the expensive part of planning.
Discussion-Write explores the code more than any other role, so the same cost applies there.

## What is being benched

The writers hand batches of independent lookups to fresh Sonnet subagents that return only what is relevant, with paths and line ranges, and read only what those lists name.
The writer is encouraged to work this way by its stencil, not commanded.
Task `explore-subagent-model` has landed (2026-11-02): the Discussion-Write stencil now carries the guidance the Plan and darn stencils already carried, and the subagent model is the `subagent_model` key of `loom.yaml` (default `sonnet`), read by all three stencils.
A fork does not serve for lookups: it inherits the writer's whole context and saves nothing.

## Research

- From the Plan-Write and Discussion-Write transcripts of landed tapestry runs, count the writers' exploration turns, their context and cost, and how many could have gone to parallel explore agents up front.
- Bench on one or two replays (the `plan-split-bench` inputs) for both roles: the writer allowed subagents but given no stencil guidance, against the writer with the guidance.
  Compare whether the guidance changes what the writer does, and cost, turns, peak context and review findings.
- Name the risk: an explore agent's list may leave out what the writer would have seen in the code, so count review findings that trace to a missed fact.

## What the bench writer did (orch, 2026-11-01)

The `psb-rv-plan` writer ran on `opus[medium]`; its Agent calls were one fork writing the plan and the `twill-plan` lens reviews, none for lookups.
No run denies the Agent tool any more, so a writer may start a fresh subagent.

## Timing

The guidance now exists for both roles, so the bench no longer waits on `explore-subagent-model`.
The "no guidance" arm needs the guidance removed from a stencil in the bench's own worktree, since the landed stencils all carry it.
