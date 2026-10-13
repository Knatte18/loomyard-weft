# Logging cleanup: right levels, no per-tick noise

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

## Inventory (2026-10-13)

Full report: `.scratch/logging-inventory.md` in the hub prime, with a verdict and reason for each of the top 30 messages.
Data: 30,111 level lines in 647 trace files (8.8 MB) across the hub prime and live pairs; 89 % WARN, 11 % INFO, no DEBUG or ERROR.

- `reed: resize re-apply failed` (`internal/reedengine/watchloop.go:437`): 83.8 % of all lines, WARN on every failing tick with no dedupe; one file under `_board` holds 26,486 lines (7.7 MB), the same tmux layout error thousands of times per session. The repeating failure itself may be a bug worth a look, beside the logging.
- `claudeengine: could not read the session name` (WARN, 3.7 %): polled before the transcript exists; Debug, or once per session.
- `orch: injection held` (INFO, 1.6 %): the dedupe fails because 409 of 494 reasons embed the changing draft text; key it on a constant reason, and stop logging the operator's draft text.
- `battenshed: producer stuck` (WARN, 1.4 %): normal polled waits; Debug, WARN only for a real block.
- `reed: watchdog lock already held, exiting` (INFO, 1.1 %): 324 short-lived processes each open a full trace file; Debug, ideally no trace file at all.
- `battenshed: session state at notice` (INFO, 0.6 %): shadow logging; Debug.
- `battencli: create worktree` dumps the whole mutation struct, duplicating `fabric: mutation`.
- Everything below rank 12 is lifecycle logging the policy asks for.

Fixing the six main items removes about 91 % of lines (about 45 % outside the one pathological file).

Test noise: the default sink is stderr at Warn, and 38 packages with Warn or Error call sites have no test that redirects the logger, so their WARN lines leak into verify logs. One `logger.SetOutput(io.Discard)` in `tmuxkit.Main` (called from `TestMain`) would silence them; no verify log survived to count.

Related: `run-log-archive` (keeping pair logs makes the noise cost more).
