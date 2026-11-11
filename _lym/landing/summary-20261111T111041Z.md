# Record a dead step loop's exit status and name its in-flight step

A `loop-exited` halt now records the loop's exit status, or the signal that killed it, even when the process that spawned the loop is gone.
It also names the interrupted step's trace, stderr and trace copy after that step moved the status file itself.
Before, these halts read `loop exit: not recorded`, and their `trace_file` often pointed at `loop.log` (issues #88, #89, #91 and #102).

## Why the exit went unrecorded

Only the waiter that spawned a loop wrote the loop's exit mark, from a goroutine reaping the loop process.
The waiter of a Go-driven run runs inside the driver process.
When the driver handed over to a successor or was restarted while the loop ran, no process was left to reap the loop.
The loop's later death then left no exit status, and the dead-loop stop could only report `loop exit: not recorded`.

## What changes

- **Loop keeper** (`internal/shedverbs/loopkeeper.go`).
  The waiter now spawns a keeper, `step --until-stop --loop-keeper=<id>`, in place of the loop.
  The keeper starts the loop process with `--loop-detached=<id>` in a session of its own and waits on it.
  When the loop process ends, the keeper writes `lym shed: loop <id> process <pid> exited: <status>` to the loop log, through the log file it already holds open.
  The status is either the exit status or the signal, such as `signal: killed`.
  The keeper catches SIGINT, SIGTERM and SIGHUP and keeps waiting, so it lives exactly as long as the loop.
  `LoopSpec.OnStart` also runs in the keeper, so loom and batten register it under the run's slug with the role `loop`, and `runproc.End` waits for it.
- The waiter's own reap now marks `keeper process <pid> exited: ...`.
  The start handshake is otherwise unchanged: the waiter treats the loop as starting until its spawn, now the keeper, exits.
- `shedverbs.DetachedLoopID` reads the loop id from either hidden flag.
  `shedcli` and `loomcli` use it in `reportArmFailure`, so an arming refusal in the keeper's pre-run still reaches the waiter as the loop's envelope.
- **In-flight step after a moved status** (`deadLoopStop`, `loopExitStop`).
  A child step that skips gated rows appends history and moves `current_producer` before it runs the next row.
  The dead-loop stop used to treat any moved status file as "no child in flight", so it dropped the trace, the stderr excerpt, `loop.stderr_path` and `loop.trace_copy`.
  It then handed the loop log to the halt note as its trace file.
  The stop now treats the pid file's child record as in flight, since the loop clears that record once a child exits.
  The detail reads `step A was in flight under trace T, and the status file now reads B` when the status moved.
  `loop.status_moved` keeps its meaning.
- Package docs (`shedverbs/doc.go`, `loomcli/doc.go`), the batten loop-spec comment, `shedrun.LoopLog`'s doc and `PATTERN-shed-driver`'s file list describe the keeper.

## Findings on the issues' other questions

- **Missing envelopes for the Drift-\* rows (#88).**
  A child step's stdout reaches the loop log only after that child exits.
  The child that died had skipped the gated Drift rows within its one step and was then running Darn-Write.
  So the history listed those rows while the loop log held no envelope for them, and the stale `trace_file` came from the moved-status rule fixed above.
- **Friction-Reflect ending normally (#89).**
  A reflection session that ends, or is closed after filing, ends its child step, which the loop reports as a step result or as an `exited` interrupt.
  A `loop-exited` stop needs the loop process itself to die, so a normally ending reflection cannot produce one.
  No change to Friction-Reflect's outcome handling was made.

## Design decisions

- The keeper is a second process per loop, not a mark the loop writes itself.
  A SIGKILL, an OOM kill or a runtime crash outside the stepping leaves the dying loop no chance to write, and only a parent can read its exit status.
  The keeper is registered through `OnStart` and catches the terminating signals.
  A teardown therefore ends it through the loop's end, and its mark is written before `runproc.End` returns.
  The mark goes through the inherited log descriptor, so a removed worktree is never recreated.
- `loop exit: not recorded` remains the text when no mark exists.
  With a keeper this now means the keeper died with the loop, as on a machine restart, or the loop came from a build that predates the keeper.
- Two asks of #102 are left out:
  a bound on the shuttle load-turn retry after `server_error`, and the idle time between a stuck step and the next step in `lym loom status`.
  Both belong to other subsystems (shuttle's load turn, loom status), and neither affects whether a loop's exit is recorded.
  With the exit status now recorded, the next such halt names its cause, and each can be its own board entry.

## Tests

- `TestLoopIntegration_ProcessExitsAreMarked` replaces the spawn-only exit-mark test with a table:
  the waiter's spawn marks its keeper's exit, the keeper marks the loop's exit status, and the keeper marks `signal: killed` for a SIGKILLed loop.
  The keeper rows also check that `OnStart` runs once.
- `TestLoopIntegration_ChildTreeDiesWithLoop` kills the real waiter, then SIGTERMs the real loop.
  It now waits for the keeper's `process <pid> exited: signal: terminated` mark and checks that the dead-loop stop's detail carries it after the signal mark.
- The dead-loop table in `loopwait_test.go` gains a row where the in-flight child moved the status: trace file, trace copy, stderr path and stderr excerpt are all named.
  A row whose child had exited still names none of them.
- The waiter fakes in `loopwait_test.go` and the helper roles in `loop_integration_test.go` map the keeper flag to the loop they stand in for.

## To check by hand

- Run a Go-driven loom run, rebuild and deploy lym while a step is in flight so the driver hands over, then kill the loop with `kill -9`.
  The next `lym loom resume` halt should read `loop exit: lym shed: loop <id> process <pid> exited: signal: killed`.
