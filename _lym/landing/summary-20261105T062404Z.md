# A loop that ends without an envelope says how it ended

When the detached `--until-stop` loop disappears without writing its envelope, the waiter's dead-loop stop marks the run failed with the cause `loop-exited`.
Until now that stop's detail, which is also the cause the halt note records, named only the loop log and the in-flight step.
It did not say whether the loop crashed or was killed from outside.
Its way forward told the reader to open `loop.trace_copy` and `loop.stderr_path`, but the stop left both empty.
This change puts the exit cause into the detail and fills both paths.
It also closes the one path inside the loop's own code that could end it without an envelope.
All changes are in `internal/shedverbs`.

## What the dead-loop stop now reports

The detail of a dead loop, and of a spawned loop that exited before it recorded itself, now carries:

- `loop exit:` followed by the marks lym wrote about that loop's process: its pid and exit status (`exit status 2`, `signal: killed`), and any terminating signal the loop recorded before it ended.
  When there are none, it reads `loop exit: not recorded`, as when a kill also ended the waiter that spawned the loop.
- `loop log:` followed by an excerpt of that loop's other lines in the loop log: a Go crash's first lines when there is one, otherwise the last lines.
- `step stderr:` followed by an excerpt of the in-flight step's captured stderr, read the same way.

The loop log is appended across every loop of the run, so the excerpt reads only the lines after this loop's starting mark.
A log with no starting mark for the loop, such as one written by an older binary, yields only the `loop exit:` clause, so an older loop's crash is never blamed on this one.
`loop.stderr_path` now names the recorded step's captured stderr, and `loop.trace_copy` a copy of its trace files when it has any.

## How the evidence gets into the loop log

- `spawnLoop` writes `lym shed: loop <id> starting` before it starts the process.
  After it reaps the process, it writes `lym shed: loop <id> process <pid> exited: <status>`.
- The loop watches for SIGINT, SIGTERM and SIGHUP for its whole life.
  When one arrives, it writes `lym shed: loop <id> received signal <name> and ends without an envelope`, then ends the process by that signal: it restores the default action and sends the signal again, or exits 1 where the platform cannot.
  `LoopSpec.Signals` and `LoopSpec.Raise` are the seams; nil selects `os/signal` and the re-raise.

## The path that ended without an envelope

`runLoop` has no return after its pid file is written that skips the envelope.
The loop could still end without one in three ways: a panic while stepping, a signal, or the machine failing.
That holds between Drift-Await's done step and Webster's start as at any other point.
The halted run's record says Webster was already in flight, so its child had started.

- **Panic.** The stepping loop now lives in `stepUntilStop`, which recovers a panic.
  It kills the in-flight step tree, logs the stack at Error to the loop's trace file, and returns an interrupted stop with cause `loop-panicked: <value>`.
  The stop then follows the usual interrupted path: the failed write, `AfterInterrupt`, `OnExit` and the envelope.
- **Signal.** The loop records the signal and ends without an envelope, as before.
  The waiter's dead-loop stop then reports it with the signal named.

## Design decisions

- **A signal writes no envelope from the loop.**
  SIGTERM reaches the loop from `runproc.End` on a cancel, a pair teardown or a reap.
  The same `End` signals the step child too, and the run's waiter or driver as well.
  Writing the failed stop and running `AfterInterrupt` inside the dying loop's kill grace would file halt notes for deliberate cancels, and could race the teardown's mark.
  The loop therefore only records the signal, and the existing dead-loop stop, run by the next waiter, stays the one place that writes the failed stop.
- **The exit status is recorded by the spawning waiter, in the loop log.**
  Only the process that spawned the loop can read its exit status.
  The loop log is where the dead-loop stop already points, and it is where the Go runtime writes a crash, so the marks sit beside that output.
  The loop id inside each mark separates loops.
  The `Spawn` seam's signature is unchanged; only the production `spawnLoop` takes the loop id.
- **The root cause of the reported Webster halt is not identified.**
  The halt note carried no evidence that could identify it, and nothing in the loop's code ends it without an envelope.
  The next such halt will name the signal or exit status.

## Tests

- `TestUntilStop_DeadLoopStopsLoopExited` gains a row with a loop log holding an older loop's crash and the dead loop's own crash, signal and exit marks, plus a step stderr.
  The row pins the exact cause, which reads only the dead loop's lines.
  Every row now checks `loop.stderr_path`, and the existing rows pin `loop exit: not recorded`.
- `TestLoop_InterruptedChildFillsTheErrorForm` adds a panicking step.
  It checks the `loop-panicked` stop, the failed status, the written envelope and the removed pid file.
- `TestLoop_TerminatingSignalIsMarkedInTheLoopLog` checks the signal mark and the raise through the seams.
- `TestLoopIntegration_ChildTreeDiesWithLoop` now ends the real loop with SIGTERM instead of SIGKILL and checks that the next waiter's detail carries the loop's signal mark.
- `TestLoopIntegration_SpawnMarksTheLoopsExit` spawns `/bin/sh` through `spawnLoop` and checks the exit mark and the stderr excerpt.
