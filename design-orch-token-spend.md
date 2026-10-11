# Cut the orch's token spend

`orch-token-spend` · Note · enhancement, orch, loom

## Seen (operator, 2026-10-11)

The hub orchestrator is now the largest token spender, ahead of Burler and Webster, and frequent distills have not cut it much.
Measured over 2026-10-10 and 2026-10-11, orch spend follows its turn count: a distill shrinks the context, but it regrows within the hour.
PR-Gate and parent-review forks are a small share of the spend, while batten and agent notices drive about 800 orch calls in those two days, so the gain is fewer orch wake-ups.

## Wanted

Cut the orch's token spend, chiefly by waking it less often.

## Means: an alignment agent in loom (operator, 2026-10-11)

The board is the manifest: it shows every run and what it is doing, and every run's discussion record is readable.
No separate manifest is kept; a hand-kept one is a chore and goes stale fast.

A cold-start alignment agent runs inside loom at the review points (PR-Gate, parent review), reads the board and the discussion and plan records of the tasks in flight, and checks that the run is aligned with them.
Its sources need no upkeep: board entries and notes, the in-flight runs' records, and what has landed on `main` since the run's base.

## Points raised in discussion

- The agent cannot see an operator decision that lives only in the orch's chat, so the orch writes every such decision into the board body it concerns.
- Parent review is also a sync point: when two overlapping tasks run at once, the orch has held both at parent review and written their reviews together so their discussion records agree. A per-run agent sees the overlap but cannot hold the other run. One shape: the agent approves when it finds no overlap with a run in flight and wakes the orch only when it does, so the orch can hold and align both.
- Open: whether in-flight runs' records need a `lyx` read verb so an agent in one pair reads another pair's records without reaching into its worktree.
- Open, for the operator: whether an approved PR review merges to `main` (and reaches prod at the next `update-plugins.sh`) on its own, or still waits for a short orch yes.

## Other means discussed

- Cheap models (such as Haiku) for the orch's simple agent jobs, like counting; fresh agents cost about $37 in the two measured days.
- The `orch: 1h` cache lifetime in `shuttle.yaml` never applied, since a distill does not restart the orch process; a restart applies it, worth only a few dollars over two days.
  Measured 2026-10-11 on the orch's main transcript (3,288 calls, 23 compactions): 39 cache expiries, 37 after more than 5 min idle; cache cost $228 under 5m against $225 simulated under 1h, so 1h saves about 1.6%.
  Cache reads are 80% of the cache bill, so the lever is the context size each call reads, not the TTL.
- The operator is also weighing running the orch on Sonnet, since most orch turns run `lyx` commands rather than reason deeply; cache reads cost the same on Opus 5.5 and Sonnet 5.5, so it saves about a quarter of the main session's spend.
