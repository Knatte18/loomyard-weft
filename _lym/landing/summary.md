# Name why a shed loop died unrecorded; drop a false shuttle disagreement

Issue #150: `lym shed step … --until-stop` halted `interrupted-stop` with `loop … ended without an envelope; loop exit: not recorded`.
Before that, the step's stderr carried `shuttle: session state disagrees … loop=held state=busy cause=background stands=loop` again every 7 to 8 minutes.
This change answers the issue's three asks.

## The repeated warning was no real disagreement

The webster strand ended turns with output files missing while background shells it does not await still ran.
The wait loop holds such a turn end (`held`), because it does not wait on unawaited shells.
The session fold reads the same turn end as `busy` with cause `background`, because the shells are still outstanding.
Both readings are correct.
The fixed mapping in `loopAgreesWithState` did not list that pair, so the shadow logged it as a disagreement.
The session woke after each shell completed: the turn start returned both sides to agreement, and the next held turn end logged the warning again.
That cycle produced the warnings at intervals.

`loopAgreesWithState` now accepts `held` beside busy on background work, as it already accepted `done` beside it.
The warning no longer fires for this pair, so it resolves itself instead of repeating.
Every other disagreement still logs once per pair, and the loop still never acts on it.
Nothing in the evidence ties these warnings to the loop's death:
the shadow is log-only, and the loop ran in another process.

## Every way a loop ends without an envelope now leaves a reason

Before this change, the loop logged some of its own early exits only to its trace, never to the loop log the waiter's stop reads.
Those exits are a lock another loop holds, a refusal to step, an uncreatable directory and an unwritable pid file.
It also exited without a loop-log reason when it could not write its envelope.
`runLoop` now marks each of these in the loop log as `lym shed: loop <id> ends without an envelope: <reason>` or `… could not write its envelope: <error>`.
The marks join the existing marks for a signal, the keeper's exit status and a Go crash.

No process can mark an exit when both the loop and its keeper die by a signal they cannot catch, or when the machine stops.
That was the issue's case, and the old detail could not tell those causes apart.
The pid record now carries `started_at`, the loop's wall-clock start.
When no exit mark exists, the dead-loop stop compares that start with the machine's boot time from the new `proc.BootTime`:
- a boot after the start reads `the machine booted at <boot>, after the loop started at <start>, so a stop of the machine ended it`;
- a boot before it reads `… so a kill of the loop and its keeper that no process can catch ended it, as by SIGKILL or the out-of-memory killer`;
- either time unknown keeps the old wording, which names both causes.

`proc.BootTime` reads the `btime` line of `/proc/stat` on Linux.
On Windows it subtracts `windows.DurationSinceBoot` from the wall clock, so time asleep counts toward uptime.
`LoopSpec.BootTime` is the test seam; nil selects `proc.BootTime`.

## Design decisions

- No timer or escalation for other disagreements.
  The shadow is log-only, by design, and the only disagreement the issue showed was this false pair.
  Fixing the mapping meets the "self-resolve" ask without giving the shadow a decision role.
- `started_at` is optional (`omitzero`).
  A record written by an older binary reads as an unknown start, and the stop then falls back to the old wording.
- The boot-time comparison runs only when the reader finds no exit mark.
  A recorded mark always outranks inference.

## Tests

- `TestWait_LogsSessionStateBesideItsClassification` gains a row: a gated turn end held with unawaited shells running logs `busy/background held` and no disagreement.
- The dead-loop cause table in `loopwait_test.go` gains rows for a boot after the start, a boot before it, and a known start with an unknown boot.
- `TestLoop_EndWithoutAnEnvelopeIsMarkedInTheLoopLog` covers a refusal and an unwritable envelope path.
  The second row also checks that the pid file records the loop's start.
- `TestBootTime` compares `proc.BootTime` with `/proc/uptime`.
