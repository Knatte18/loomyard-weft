# A dependent task plans ahead from its dependency's record and plan, and waits only to build

`pipelined-chain-planning` · Note · feature, loom, batten, shed

## Seen (operator, 2026-11-28)

A chain of tapestry tasks where B depends on A runs fully in series: B's Discussion and Plan wait until A has landed, though the plan writer is the slowest row.
The six serial parts of project `fabric-core` are the case at hand.

In Millhouse, which LoomYard grew from, the operator sped such chains up by hand, with success:

- Once A's discussion record was approved, B's Discussion started, told to build on everything A had decided.
- Plan writing for B likewise started before A was done, from A's record.
- When A had landed, B merged A's changes, and a small fixer read the code changes and updated B's discussion and plan for what had moved.
- Only then did B's builders start, so the builders were the only part that waited for A.

## Idea

Let lym do this systematically, since keeping it straight by hand is hard:
a task whose dependency has an approved record (and later an approved plan) may run its own Discussion and Plan rows ahead,
reading the dependency's record and plan as given; it then parks before its build rows until the dependency lands,
merges the landed change, runs a small re-align row that updates its record and plan for what the landing changed, and builds.

## Bounds

- When A rewrites a very large part of what B builds on, the plan written ahead is mostly wasted, and B should wait as today.
  The re-align row needs a measure of how much changed, and a threshold past which B plans again from scratch.
- Not for now (operator, 2026-11-28).
