# Card 20 — the orch's plan amendment route

**Prosa:**
- `contracts/stencils/orch/orch-template-role.md`

**Intent:**
D8: the orch stencil routes a design call that amends an approved plan through the Plan-Review fix loop, never a card edit.

The "design call the child cannot make" bullet and the "Fixing a run" section gain: a design call that amends the approved plan is recorded with `lyx loom decision add <slug> --by parent … --amends-plan`, worded as a concrete change to named cards, each named by slug as well as number, then `lyx loom goto --to Plan-Burler-Review` and `lyx loom resume` in the task worktree; never by editing a card.
The PATTERN-change sentence under "An operator decision made in chat" notes that once the plan is approved the parent also passes `--amends-plan`.
A spent Plan-Review bounce budget pauses the judge for `lyx loom resume`, which bounds the orch's repeats by its own judgment.

The stencil is prose an agent reads; no test asserts its text.
