# Orch manifest in Bolt, cold reviewer replaces PR-Gate

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
