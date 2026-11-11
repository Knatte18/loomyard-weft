# shedverbs: record how an `--until-stop` loop's process ended

A `lym shed step --until-stop` loop that ended without an envelope halted with `loop exit: not recorded`.
Nothing told a crash, a kill and a clean exit apart, and the halt note did not always name the step that was in flight.
After this change the halt note carries the loop process's exit status or terminating signal, and names the in-flight step, its trace and its stderr.

Closes #88
Closes #89
Closes #91

## What changes

- The waiter no longer spawns the loop process directly.
  It spawns a keeper, started through the new hidden `--loop-keeper=<id>` flag on `step`.
  The keeper starts the loop process (`--loop-detached=<id>`) in a session of its own, waits for it, and writes a mark to the loop log with the process's pid and exit status or signal.
  The keeper catches the terminating signals and keeps waiting, so a signal that ends the run's processes ends the keeper only through the loop's end.
  A loop process that does not start gets a mark naming the start error.
  See `internal/shedverbs/loopkeeper.go`.
- The waiter's own mark now names the keeper's exit (`keeper process N exited: ...`).
  The mark of the loop process is the one `loopExitEvidence` reads.
- The dead-loop stop waits a bounded time (`keeperMarkLooks` looks, `keeperMarkPoll` apart) for the keeper's mark.
  The keeper writes it after the loop has released its lock, so a waiter that sees the free lock can look first.
  After the last look it reports the exit as not recorded.
- The dead-loop stop names the in-flight step, its trace and its stderr whenever the pid file records a child.
  Before, it did so only while the status file still read that step.
  When the status file has moved to another producer, the detail says so.
  A child moves the status file itself when it skips gated rows.
- The halt note's `trace_file` points at the in-flight child's first trace file, and falls back to the loop log only when no child was in flight or it has no trace file.
- `LoopSpec.OnStart` also runs once in the keeper after it has started the loop process, so the loop is registered in the hub's owner record under both processes.
  `DetachedLoopID` makes an arming refusal in the keeper or in the loop process reach the waiter as the loop's envelope.
- The doc comments in `shedverbs`, `shedrun`, `loomcli`, `shedcli`, `battencli` and `pattern/PATTERN-shed-driver.md` follow the keeper.

## Design decisions

- Recording the status in a keeper, rather than in the waiter, keeps the mark written when the waiter is already gone.
  The waiter is not always there to reap the loop: it can be ended or time out while the loop runs on.
- The keeper outlives the loop process by design, so its exit is a separate mark from the loop process's exit.
- Scope: issue #102 also asks for a bounded load-turn retry on `server_error` and for idle time in `lym loom` status.
  Neither is in this change; both are board task `load-turn-retry-cap`, which waits on this one.
  Issue #102 is therefore not closed here.

## To check by hand

- Issue #89 asks whether a Friction-Reflect session that ends normally, or is closed after filing, can be reported as an interrupted loop exit, and if so to treat the step as done.
  The diff does not change that handling, so the question stays open.
- Issue #88 asks why the loop log lacked envelopes for the Drift-* steps that the history listed.
  The diff does not investigate it.
- Issue #102's ask for the reason a loop stopped (retry cap, signal, crash) is met only for the exit status and signal, as the first design decision above says.
  The unrecorded-asks decision was left open by the review, and the scope decision above is what records it.
- Starting the loop through a keeper is covered by the arming and keeper tests in `internal/shedverbs`.
  The keeper's refusal to arm with the keeper flag set is not tested: `keeper-flag-arming-refusal-untested`, in `internal/shedverbs/loop_arming_integration_test.go` and `loopkeeper.go`.
- The keeper surviving a terminating signal sent to the run's processes is not tested end to end: `keeper-signal-survival-untested`, in `internal/shedverbs/loop_unix_integration_test.go` and `loopkeeper.go`.
- Windows behaviour of the keeper's detached start, and of the keeper's breakaway retry in `spawnLoop`, was not run.
