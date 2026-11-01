# Plan writer hands its code lookups to forks and keeps the overview

`planner-fork-lookups` · Note · research, loom

## Seen (orch, 2026-11-01)

In a sample of 28 sessions (`.scratch/batching-breach.md` of the batching study), the single largest cost was the Plan writer of `run-verbs-shared` in `plan-split-bench`: 31 chained grep turns at 390k to 530k context and 15 one-file-per-turn fix turns at 630k to 668k.
Every lookup the writer makes is paid at its whole context, so exploration late in a large session is the expensive part of planning.

## Question (operator, 2026-11-01)

Can the Plan writer hand its code exploration to forks, which answer a batch of questions and return only conclusions, so the writer keeps its overview but stops paying its full context for each grep?
The writer must keep the overview of the whole plan; only the lookups move.

## Research

- From the Plan-Write transcripts of landed tapestry runs, count the writer's exploration turns, their context and cost, and how many could have been asked as one batch of questions up front.
- Try fork-based exploration on one or two replays (the `plan-split-bench` inputs), and compare cost, turns, peak context and Plan-Review findings with the bench's Plan-Write arms.
- Name the risk: conclusions from a fork may miss what the writer would have seen in the code, so count review findings that trace to a missed fact.
