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

## Design (operator, 2026-11-28)

Tapestry only: darn has no separate plan row to run ahead.

1. B depends on A.
2. When A's plan is approved, B is released.
3. B runs Discussion and Plan as usual, Plan Perch included, with A's record and plan as input.
4. After B's plan is approved, a new producer row waits for B's dependencies: B parks there until A has landed.
5. When A has landed, that row merges B's parent into B, bringing in A's change,
   and a fixer updates B's decision record and plan for what A's change brought.
   This stays small when A kept to its own plan.
6. B then goes on to Webster as today.

The merge in step 5 also brings in whatever else landed on the parent, not only A.
That is no new cost: Parent-Align already meets the same landings later today.
Running the chain in a beam of its own keeps the parent quiet, so the merge in step 5 brings in little beside A.

## Bounds

- The choice is made up front, before B plans ahead, never after: a plan written ahead and then thrown away is the waste to avoid.
  From A's record and plan, estimate how much of what B builds on A will change;
  when A is estimated to change too much of it, B does not plan ahead and waits as today.
- A run parked in the wait row of step 4 does nothing useful, so it must not hold one of `beam.max_runs`'s run places.
- Each link planned ahead of landed code adds error to the plans, so the cascade is kept short.
- Open: when B's plan counts as done for C to start.
  Proposed: only when B has passed step 5, so C plans from a plan re-aligned against A's landed code and the chain runs at most one link ahead.
- Not for now (operator, 2026-11-28).
