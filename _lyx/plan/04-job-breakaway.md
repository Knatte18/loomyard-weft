# Card 4 — job-breakaway

**Edit:**
- `internal/proc#createKillOnCloseJob`

**Extends:**
- `internal/proc/doc.go`

**Intent:**
On Windows, let a process started inside a step leave the step's jobs when, and only when, it asks to (D4).

`createKillOnCloseJob` sets `JOB_OBJECT_LIMIT_BREAKAWAY_OK` beside `JOB_OBJECT_LIMIT_KILL_ON_JOB_CLOSE`.
It builds both the unnamed per-child job `StartTied` gives a step and the named job `HoldNamedJob` gives a loop, so both admit a `CREATE_BREAKAWAY_FROM_JOB` start, which `proc.DetachBreakaway` asks for.
Without the limit Windows refuses that start with `ERROR_ACCESS_DENIED`, which `IsBreakawayRefused` already reads.

What this lets through, and its bound: a process started with the breakaway flag outlives its step's and loop's jobs.
Only `proc.DetachBreakaway` sets the flag, and its non-test callers are `spawnLoop` (reached only from the `--until-stop` waiter outside any step, D4) and the capture spawn of card 5-capture-run.
A process started without the flag stays in its job, so every other descendant of a step still dies with it, and kill-on-close is unchanged.

`doc.go` gains one sentence: the per-child and named jobs admit breakaway, so only a `DetachBreakaway` start leaves them.

The function is Windows-only (`namedjob_windows.go`); no gate of this hub runs Windows, and `cmd/lyx`'s cross-compile test builds the file for `windows`.
The behavior is proved on Windows by the sandbox hub's run under board note `csharp-sandbox-hub`, outside this task (Scope, Out).

**ImpactSummary:** One limit flag on the Windows job both `StartTied` and `HoldNamedJob` create; Linux code untouched.
