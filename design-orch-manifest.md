# Alignment agent in loom instead of orch at PR-Gate

`orch-manifest` · Note · feature, orch, loom, board

## Seen (operator, 2026-10-11)

The hub orchestrator is now the largest token spender, ahead of Burler and Webster, and frequent distills have not cut it much.
The orch's value at PR-Gate and parent review is that it knows everything that has happened and is planned, which a cold Bouncer lacks.

## Idea (operator, 2026-10-11)

- Keep a manifest in Bolt that the orch updates as it goes: the context that otherwise lives only in the orch's chat.
- A cold-start agent reviews at PR-Gate (and similar points) with the manifest as its basis, so loom can spawn it as a row of its own and the orch drops out of that loop; notices then go to the orch only for landings, stops and real escalations.

## Points raised in discussion

- The manifest holds only what `lyx` cannot derive: direction, cross-task operator decisions, patterns seen across runs, approaches on hold. Runs in flight, dependencies and touched files the reviewer reads itself; a single task's intent stays in its board body.
- Pairs reach Bolt only through `lyx`, so the manifest needs a read verb, and dated entries.
- A stale manifest gives a confident wrong review; the orch prunes it at each distill, and the orch note could fold into it.
- Open, for the operator: whether an approved review merges to `main` (and reaches prod at the next `update-plugins.sh`) on its own, or still waits for a short orch yes.
- The operator is also weighing running the orch on Sonnet, since most orch turns run `lyx` commands rather than reason deeply.

## Revised idea (operator, 2026-10-11): an alignment agent instead of a manifest

A hand-kept manifest is a chore and goes stale fast.
The board already shows every run and what it is doing, and every run's discussion record is readable.
So a cold-start agent runs inside loom, reads the board and the discussion (and plan) records of the tasks in flight, and checks that the run is aligned with them.

- Its sources need no upkeep: board entries and notes, the in-flight runs' discussion and plan records, and what has landed on `main` since the run's base.
- What it cannot see is an operator decision that lives only in the orch's chat, so the orch writes every such decision into the board body it concerns, as it mostly does today.
- Open: whether in-flight runs' records need a `lyx` read verb so an agent in one pair reads another pair's records without reaching into its worktree.
- Measured 2026-10-11 (`.scratch/orch-token-breakdown.md`): orch spend follows its turn count; PR-Gate and parent-review forks are a small share, while batten and agent notices drive about 800 orch calls in two days, so the gain is fewer orch wake-ups.
