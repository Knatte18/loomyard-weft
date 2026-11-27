# Class transient failed halts as transient-stop; log parent lock waits

A step whose producer error carries a transient class, such as Finalize's read of GitHub's branch rules failing on a DNS lookup, persists state `failed` with `transient=github-api`.
The driver's rule table already re-steps such a stop with waits.
The halt note that step left still named it `producer-hard-failure`,
so the friction reflection read an outage that cleared at once as a defect.

## Changes

- `loomengine.ClassifyHalt` takes a `transient` flag and returns the new kind `transient-stop` for a failed halt that carries one.
  An interrupted stop and a build skew keep their own kinds.
- The failed-halt note in `internal/loomcli/halt.go` reads the transient class from the status file, so the note names `anomaly: transient-stop`.
- The reflection template drops a `transient-stop` halt, unless the notes show re-steps at one row that never cleared it, or a failure classed transient that no retry could fix.
- Finalize's parent lock guard logs the wait's start and holder, each change of holder, and the wait's end.
  The end line carries both the wall-clock time waited and the awake time,
  so a nine-hour gap like the one in the report shows whether the lock was held that long or the host was suspended.

## Design decisions

- The status file keeps state `failed` with its `transient` class, and no new shed state is added.
  That class is already the retryable marker: the driver re-steps a git-transport or GitHub API stop three times with waits before it hands back.
  A new state would move every reader of `failed`, the run state machine's halted reading and the board status among them.
  Retrying in place of failing is the scope of the separate task on landing pushes and reads.
- A transient halt still writes its note, under its own kind, so the outage stays on record and the reflection can tell it apart from a defect.

## Tests

- `TestClassifyHalt` gains rows for a transient failed halt, a transient interrupted stop and a transient non-failed state.
- `TestStep_ProducerErrorWritesTheHaltNoteAndNoReflection` gains a row that steps a producer returning a GitHub API error marked transient, and expects `anomaly: transient-stop` in the note.
  On the merge base the same note reads `producer-hard-failure`.
- The wait log lines are read only by people, so no test pins them.

## Merge danger

**Door:** two-way
**Blast radius:** loom
