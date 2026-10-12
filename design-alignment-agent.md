# Alignment agent in loom checks a run against the board and runs in flight

`alignment-agent` · Task · enhancement, orch, loom · running · Plan-Burler-Fix

Split out of note `orch-token-spend` (2026-10-11), whose other means went to `orch-owns-the-loop` (batten lands itself, what wakes the orch), `merge-review`, `orch-split` and `task-run-settings`.

## Seen (operator, 2026-10-11)

The hub orchestrator is the largest token spender, ahead of Burler and Webster.
Measured over 2026-10-10 and 2026-10-11, orch spend follows its call count: every call reads the whole context, a distill shrinks it but it regrows within the hour, and batten and agent notices drove about 800 orch calls in those two days.
A 1h cache TTL saves only about 1.6% (measured 2026-10-11), since cache reads are 80% of the cache bill.

## Idea: an alignment agent in loom (operator, 2026-10-11)

The board is the manifest: it shows every run and what it is doing, and every run's discussion record is readable.
No separate manifest is kept; a hand-kept one is a chore and goes stale fast.

A cold-start alignment agent runs inside loom at the review points (PR-Gate, parent review), reads the board and the discussion and plan records of the tasks in flight, and checks that the run is aligned with them.
Its sources need no upkeep: board entries and notes, the in-flight runs' records, and what has landed on `main` since the run's base.

## Points raised in discussion

- The agent cannot see an operator decision that lives only in the orch's chat, so the orch writes every such decision into the board body it concerns.
- Parent review is also a sync point: when two overlapping tasks run at once, the orch has held both at parent review and written their reviews together so their discussion records agree. A per-run agent sees the overlap but cannot hold the other run. One shape: the agent approves when it finds no overlap with a run in flight and wakes the orch only when it does, so the orch can hold and align both.
- Open: whether in-flight runs' records need a `lyx` read verb so an agent in one pair reads another pair's records without reaching into its worktree.
- Open, for the operator: whether an approved PR review merges to `main` (and reaches prod at the next `update-plugins.sh`) on its own, or still waits for a short orch yes.
  Operator comment on this (2026-10-12): the point of this note, and of several others from 2026-10-11, is to disturb the orch less; far too much goes to the orch, and most of it should stay inside loom and the task's worktree.
- Open, for the operator (2026-10-11, undecided): when to start this and `merge-review`, now that `burler-split-budget` has landed. This note changes parent review, which `parent-review-wait` is building, so the orch's suggestion was to start `merge-review` now and this one after `parent-review-wait` lands; the operator wants to discuss it later.
- The operator has weighed running the orch on Sonnet, since most orch turns run `lyx` commands; the operator also finds Sonnet generally weaker than Opus.
