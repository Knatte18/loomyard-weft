# A dependent task plans ahead from its dependency's record and plan, and waits only to build

`pipelined-chain-planning` · Note · feature, loom, batten, shed

Depends on: [`recipe-build-from-sketch`](design-recipe-build-from-sketch.md), [`release-order-queue`](design-release-order-queue.md)

## Seen (operator, 2026-11-28)

A chain of tapestry tasks where B depends on A runs fully in series: B's Discussion and Plan wait until A has landed, though the plan writer is the slowest row.
The six serial parts of project `fabric-core` are the case at hand.

In Millhouse, which LoomYard grew from, the operator sped such chains up by hand, with success:

- Once A's discussion record was approved, B's Discussion started, told to build on everything A had decided.
- Plan writing for B likewise started before A was done, from A's record.
- When A had landed, B merged A's changes, and a small fixer read the code changes and updated B's discussion and plan for what had moved.
- Only then did B's builders start, so the builders were the only part that waited for A.

## Design (operator, 2026-11-28)

Only for the recipe that starts from a stored decision record (jacquard): darn has no separate plan row to run ahead,
and a run with its Discussion still to run never plans ahead, so B always plans from a record already approved.
The model of plan and build as two recipes sharing one standing pair was weighed and dropped:
it would leave a pair on disk with no run, against the rule that only a run has a worktree on disk.

1. B depends on A.
2. When A's plan is approved, B is released.
3. B runs Plan as usual, Plan Perch included, with A's record and plan as input.
4. After B's plan is approved, a new producer row waits for B's dependencies: B parks there until A has landed.
5. When A has landed, that row merges B's parent into B, bringing in A's change,
   and a fixer updates B's decision record and plan for what A's change brought.
   This stays small when A kept to its own plan.
6. B then goes on to Webster as today.

The merge in step 5 also brings in whatever else landed on the parent, not only A.
That is no new cost: Parent-Align already meets the same landings later today.
Running the chain in a beam of its own keeps the parent quiet, so the merge in step 5 brings in little beside A.

## One dependency list (operator, 2026-11-28)

- `depends_on` keeps one list and one meaning: B builds only after each entry in it has landed.
- When B may start planning is derived from that list, with no second field:
  a jacquard task B may start once each of its dependencies has landed or has an approved plan.
  A dependency with no plan row of its own, such as a darn task, counts only once it has landed:
  darn runs are short, and its record is too thin to plan ahead from.
  The wait row of step 4 then holds B until every dependency has landed.
- While B plans or parks ahead of a dependency that has not landed, the board marks it beside its status, such as `plans ahead of <slug>`.

## A parked run (operator, 2026-11-28)

- A run parked in the wait row of step 4 holds none of `beam.max_runs`'s run places, no tmux session and no Konsole window, so it leaves the KDE tiling alone.
  Its pair stays on disk, local; nothing it needs is trusted to origin.
- When its dependencies are done, dobby reopens its session and window and resumes it.
- A parked run whose dependencies are done is resumed before dobby starts any new task:
  it was released when it began planning, so it never rejoins the back of the ready queue.

## Bounds

- The choice is made up front, before B plans ahead, never after: a plan written ahead and then thrown away is the waste to avoid.
  From A's record and plan, estimate how much of what B builds on A will change;
  when A is estimated to change too much of it, B does not plan ahead and waits as today.
- Each link planned ahead of landed code adds error to the plans, so the cascade is kept short:
  one parked run per chain. C may plan ahead only once B has passed step 5, so C plans from a plan re-aligned against A's landed code.
- Not for now (operator, 2026-11-28).
