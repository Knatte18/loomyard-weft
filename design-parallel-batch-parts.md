# Parallel batches: the shared pieces for Webster forks and Weaver

`parallel-batch-parts` · Note · enhancement, webster, fabric, loom

## Seen (2026-10-17)

The pieces parallel batches need are the same whether a batch starts as a Webster fork or as a cold Weaver agent (see `weaver`); only the spawn differs.

1. A DAG batchifier: edges from the cards' targets and `Uses:`, cards joined into batches while the batch graph stays acyclic and under a cost budget, ready batches chosen. New.
2. A worktree per batch, created for the batch and removed after it. New; may build on fabric.
3. Merge-back of each batch worktree into the task branch, with a cheap model such as Haiku resolving textual conflicts. New.
4. The verify on the merged result and its failure ladder. Exists in Webster (verify gate).
5. The batch brackets and report: `begin-batch`, `record-batch`, the changed-files guard and the report contract. Exist in Webster.
6. `plan-dag-view`, to see the graph and the batches. Note filed.

## Operator view, not a decision (2026-10-17)

- Start with these pieces, built so both Webster and Weaver can use them; Webster can run parallel forks first.
- The hard part is getting a Webster fork to work in another worktree than its master's.
- Claude Code's own worktree isolation for agents has given the operator only trouble before.

## Open

How a fork is kept inside its batch's worktree: it inherits the master's working directory and a context full of paths into the task worktree, and `lyx` commands find their anchor from the working directory.

### Orch recommendation

Bench before build: a small proof of concept in which one fork is told to work in a worktree Go created, with absolute paths, and a PreToolUse hook refuses its Edit and Write calls outside that worktree, so a slip is refused instead of landing in the wrong tree; measure whether the fork stays inside and what `lyx` commands it needs a directory flag for.
