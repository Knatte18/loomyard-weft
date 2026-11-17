# Trial: check five lym gate redundant candidates by hand after the kill-count fix

`redundant-candidates-check` · Task · research, gate, testkit · done

## Seen (orch, 2026-11-17)

`map-tooling-trial` (`a666e8b3`) found 14 candidates in `internal/boardengine` but checked none of them,
and `redundant-timeout-not-kill` (`b904a1f8`) then stopped counting a timed-out or memory-capped mutant run as a kill and added an `unconfirmed` list.
The earlier survey found 27 of 30 sampled confirmations false. Before `redundancy-sweep-existing` relies on the list, the operator wants a few candidates checked (operator, 2026-11-17).

## Idea

A trial in the task's own worktree, with its report landed as a commit:

- Build `internal/boardengine`'s map fresh with `lym gate test --map`, then run `lym gate redundant internal/boardengine` once.
  Record wall time, peak memory, and the `candidates`, `unconfirmed` and `refuted` lists against the trial's 14 candidates.
- Read five of the new candidates, picked across the list, against the tests that cover the same blocks,
  and judge each: does it assert anything its covering tests do not (absent side effects, persisted state, inputs the others do not feed)?
- Report how many of the five are true redundancies, with the assertion each false one adds.
- Each defect found becomes its own note; the trial fixes nothing and deletes no test.

## Cap

- One package, one map build, one `lym gate redundant` run, five candidates read.
- No model or effort choice: the report is written by the darn writer at its default model and effort.
