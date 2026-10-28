# Decision record — transient-halt-recovery

## Goal

A momentary host process shortage, EAGAIN from fork or exec, no longer ends a loom run hard, and an interrupted stop's halt note names its cause, its trace and the producer the status file blames.

## Scope

- `internal/gitexec`: a git the host could not start for EAGAIN is started again with a short doubling backoff before the failure surfaces.
- `internal/proc`: a process-wide record of the last EAGAIN spawn refusal, with the one EAGAIN predicate reed and gitexec share.
- `internal/reedengine`: the tmux EAGAIN retry uses the shared predicate and notes each refusal in the record.
- `internal/shedengine`, `internal/shedtransient`, `internal/shuttleengine`, `internal/shedadapters`: a new transient class for a host resource shortage, covering an error that carries EAGAIN and an agent that died shortly after the step's process saw an EAGAIN refusal.
- `internal/shedverbs` loop: an interrupted stop's cause carries the child's command and its stderr excerpt, its trace file falls back to the captured stderr, the stop blames the producer the status file names after the child exited, the failed-stop write comes before the interrupt hook, and the loop envelope carries the status file's pause reason.
- `internal/sheddriver`: a paused stop whose envelope has no reason takes the loop's pause reason.
- `internal/loomcli`, `internal/loomengine`: the interrupted-stop note records the state and error the status file holds after the stop.
- Docs: the package `doc.go` files of the changed packages.

## Decisions

### Decision: retry an EAGAIN git spawn inside gitexec

`gitexec`'s exec core starts git again when the start itself failed with EAGAIN: four tries in all, 250 ms before the first retry and doubling, each retry logged at Warn.
Rationale: every fabricengine git call goes through gitexec, so the retry covers the failing `read task patch` diff and every other git read or write at once; it mirrors reed's tmux retry.
Bound: only a start that failed with EAGAIN is retried, so a git that ran and exited is never run twice; the retry adds at most 1.75 s of waits before the failure surfaces unchanged.

### Decision: one EAGAIN predicate and a process-wide refusal record in proc

`proc.IsSpawnAgain` is the one test for an EAGAIN spawn refusal, and `proc.NoteSpawnAgain` records the time this process last saw one; reed and gitexec note each refusal they see.
Rationale: proc is the stdlib-only package both spawners already import, so neither gains a dependency, and the record lets the transient classifier see a refusal a lower layer retried past.

### Decision: a host-resources transient class

shedengine's closed class set gains `host-resources`.
shedtransient returns it for an error that carries EAGAIN, and for an error wrapping a new `shuttleengine.ErrAgentDied` when the process noted an EAGAIN refusal within the last two minutes; the seats adapter wraps `ErrAgentDied` into a chair's `died` outcome that started.
Rationale: a spawn the host refused after the retries, or an agent that died right after the host refused spawns, can plausibly succeed when the same step runs again, which is the class set's admission test.
Bound: the class lets the loop re-step such a failure once, and the driver's existing rule hands a transient that outlived its re-step to the tackler; a died agent with no EAGAIN refusal in the window, and every timeout, stay hard failures.

### Decision: the interrupted cause names the command and the stderr excerpt

A child that exited without an envelope is reported with its command line and an excerpt of its captured stderr beside the exit status: a Go runtime crash's first lines when stderr holds one, and otherwise its last lines.
Rationale: exit status 2 is a Go panic or fatal error, whose reason is on stderr, so the cause then names it in the status error, the envelope and the halt note.

### Decision: the trace file falls back to the captured stderr

When the child left no trace file, the interrupted stop's trace file is the captured stderr file.
Rationale: a child that died before its logger wrote leaves no trace, and the stderr capture is then the only file that holds what it said.

### Decision: blame the producer the status file names after the child exits

The interrupted stop's producer is the status file's current producer read after the child exited, falling back to the one read before it.
Rationale: a child that skipped gated rows moved the run on before it died, so the producer read before it names a row the history records as skipped.

### Decision: write the failed stop before the interrupt hook, and note the status as found

The loop writes the failed stop before it runs the interrupt hook, and loom's interrupt hook records the state and error the status file then holds; `ClassifyHalt` reports `interrupted-stop` for an interrupted halt whatever non-blocked state the status holds.
Rationale: a child that died before its resume write leaves a paused run paused, so a note that always says `failed` misstates it, and a child that recorded its own failure leaves that error for the note to link.

### Decision: a paused stop carries the status file's pause reason

The loop envelope carries the after-status's pause reason, and the driver takes it as the stop's reason when the envelope's own reason is empty.
Rationale: an interrupted step over a paused run leaves the run paused under its recorded reason, which the driver read as a pause with no reason.

### Decision: no change for concurrent spawns at the end of Darn-Write

lym's own spawns are not changed.
Rationale: the exhaustion was the user slice's task limit, filled host-wide by leaked MSBuild reuse nodes, which board entry `gate-robustness` owns.

## Constraints

- PATTERN-transient-stop: the new class is marked only through shedtransient at the producer and step-bootstrap boundaries, never on a gate verdict.
- PATTERN-gitrepo-client-boundary: the retry lives in gitexec's exec core, so no raw git site is added.
- PATTERN-tolerant-records: the loop envelope's new key is optional and the driver tolerates its absence.
- PATTERN-sole-parsers: the halt note keeps its format and its sole renderer, `internal/haltnote`.
- PATTERN-test-speed: tests inject the retry wait and stub the spawn; none waits on a real clock.

## Auto-mode assumptions

- A darn run holds no interview; the scope is the board entry, refined against the code as the Decisions state.
- The paused run of issue #891 was a goto-paused run whose stepped child died before its resume write, which left the status paused and its pause reason out of the loop envelope.

## Open risks

- A died agent within two minutes of an unrelated EAGAIN refusal is re-stepped once though the death had another cause; the re-step is bounded to one.
- The stderr excerpt heuristic may pick a logger line when the child wrote no Go crash.

## Acceptance criteria

- A git start that fails with EAGAIN is retried and succeeds when a later try starts; a start that keeps failing returns the EAGAIN error after four tries; a git that exits non-zero is never retried.
- shedtransient classifies an error carrying EAGAIN as `host-resources`, a died-agent error as `host-resources` only after a noted EAGAIN refusal, and its existing rows unchanged.
- An interrupted child's cause carries its command and the stderr excerpt, a Go crash's first line picked over a trailing stack.
- An interrupted stop with no trace file names the captured stderr file as its trace file.
- An interrupted stop after the child moved the run on blames the status file's current producer.
- An interrupted stop over a run the child left paused keeps the status paused, and its loop envelope and the driver's stop carry the pause reason.
- The interrupted-stop halt note records the status file's state and the anomaly `interrupted-stop`.
