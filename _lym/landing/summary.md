# Retry EAGAIN spawns and name the cause of an interrupted loom stop

A momentary host process shortage no longer ends a loom run as `failed`, and an `interrupted-stop` halt note now says what the child did and which producer it was on.

Closes #886
Closes #890
Closes #891

## What changes

- `internal/gitexec` starts a git the host refused for EAGAIN again, four tries in all, waiting 250 ms before the first retry and doubling after.
  Only a failed start is retried, so a git that ran and exited non-zero never runs twice.
  Each retry is logged at Warn, and the failure surfaces unchanged after the last try.
- `internal/proc` gains `IsSpawnAgain`, the one EAGAIN predicate, and `NoteSpawnAgain`/`LastSpawnAgain`, a process-wide record of the last refusal.
  `internal/reedengine`'s tmux retry uses the predicate and notes each refusal, and so does gitexec.
- `internal/shedengine` gains the transient class `host-resources`.
  `internal/shedtransient` returns it for an error carrying EAGAIN, and for an error wrapping the new `shuttleengine.ErrAgentDied` when the process noted an EAGAIN refusal within the last two minutes.
  The seats adapter in `internal/shedadapters` wraps `ErrAgentDied` into a chair's `died` outcome.
  The loop re-steps such a failure once; a transient that outlives its re-step goes to the tackler under the driver's existing rule.
- `internal/shedverbs` loop, for a child that exits without an envelope:
  - The cause carries the child's command line and a stderr excerpt, a Go crash's first lines when stderr holds one and the last lines otherwise.
  - The trace file falls back to the captured stderr file when the child left none.
  - The stop blames the producer the status file names after the child exited, not the one read before, so a producer the child skipped past is no longer blamed.
  - The failed stop is written before the interrupt hook runs.
  - The loop envelope carries the status file's pause reason in a new optional key.
- `internal/sheddriver` takes that pause reason as a paused stop's reason when the envelope's own is empty, so a paused run is no longer read as paused for no reason.
- `internal/loomcli` and `internal/loomengine`: the interrupted-stop note records the state and error the status file holds after the stop, and `ClassifyHalt` reports `interrupted-stop` for an interrupted halt in any non-blocked state.
  A run the child left paused is therefore no longer recorded as `failed`.

## Design decisions

- The retry lives in gitexec's exec core, so every fabricengine git call, including the `read task patch` diff that failed in #886, is covered with no new raw git site.
- `proc` holds the predicate and the record because both spawners already import it, and the record lets the classifier see a refusal a lower layer retried past.
- A died agent counts as `host-resources` only after a noted EAGAIN refusal; a died agent without one, and every timeout, stay hard failures.
- The loop envelope's new key is optional and the driver tolerates its absence.
- Not changed: how many processes lym spawns at the end of `Darn-Write`.
  The exhaustion behind #886 was the user slice's task limit filled host-wide by leaked MSBuild reuse nodes, which board entry `gate-robustness` owns.

## Check by hand

- A died agent within two minutes of an unrelated EAGAIN refusal is re-stepped once though its death had another cause; the re-step is bounded to one.
- The stderr excerpt picks a logger line when the child wrote no Go crash; read a real interrupted-stop note to confirm the excerpt is useful.
- No real EAGAIN was provoked; the tests stub the spawn and inject the retry wait.
