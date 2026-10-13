# WARN for real problems only: batten waiting is an action log

`log-level-hygiene` · Note · bug, batten, orch

## Seen (2026-10-13)

In the hub prime's trace logs (`.lyx/logs`, 10-10 to 10-13), 516 of 2339 lines are WARN:

| Count | Message | What it is |
|---|---|---|
| 415 | `battenshed: producer stuck` | Mostly normal waiting, such as `reason="inner shed run still running; checking it every 2s"` or `"child running → awaiting at PR-Gate"`, logged at each check |
| 79 | `orch: session state disagrees with the idle probe` | A real diagnostic, repeated per reading |
| 11 | `gitrepo: push refused, retrying` | A real anomaly that recovers by itself |
| 8 | `battenshed: inner shed run halted; waiting for the operator to resume it` | A pause, not a problem |

So about 80 % of WARN lines record an ordinary state as a problem: an action log, not a warning (operator, 2026-10-13).
`internal/logger/logger.go` states a level policy ("Level policy"); these lines break it.

## Idea

- Batten logs a producer's normal waiting at Info or Debug, once per change of state, never per poll; WARN only when a run is stuck in fact.
- The orch's idle-probe disagreement logs once per change, not per reading.
- A halted run waiting for the operator is Info.
- Check the other modules' WARN calls against the level policy while at it.

Related: `run-log-archive` (keeping pair logs makes the noise cost more).
