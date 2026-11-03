# Drift-Baseline halt shows the capture child error

`drift-child-error-in-halt` · Task · enhancement, drift · running · Worktree-Create

From [issue #26](https://github.com/Knatte18/loomyard/issues/26).

## Seen

Drift-Baseline halted four times in one task with "drift: the capture child ended before it recorded its start".
The real cause, a cwd refused by the anchor check, sat only as one line in the capture child's log under `_lym/drift/logs/`, so the parent learned it only after opening the log.
The halt advised `lym loom resume`, which could not change the outcome, since the same error recurred on every entry.

The cwd fault itself landed in `93eaaa45` (capture and compare start in the anchored directory, and the halt names `lym drift defer`).
This task keeps the two halt-message parts that landing did not build.

## Expected

- When a capture child ends before recording its start, the halt carries the child log's error text (its last error line, bounded in length), not only the log's path; `startFailure` in `internal/driftengine/spawn.go` is where the message is built.
- When the child's error is one a resume cannot change, such as the cwd and anchor check, the halt says so and drops `lym loom resume` as a way forward, naming only `lym drift defer` and the fix of the cause.
