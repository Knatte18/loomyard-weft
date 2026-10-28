# EAGAIN no longer ends a loom run hard; interrupted stops name a cause

A git start the host refuses for EAGAIN is retried, a host resource shortage is a transient failure the loop re-steps once, and an interrupted stop's halt note names its cause, its trace and the row the status file blames.

## Why

Issues #886, #890 and #891 report loom runs that halted `failed` during a momentary process-table shortage on the host:

- A `fork/exec /usr/bin/git: resource temporarily unavailable` in a fabric read ended the run as `producer-hard-failure` with no retry.
- A seats chair that died ten seconds after reed's tmux EAGAIN retries carried no transient mark, so the run failed instead of re-stepping.
- The sibling `interrupted-stop` notes said only `exited: exit status 2`.
  One had an empty `trace_file:`, and one blamed `Drift-Verify`, which its own history records as skipped.
- The Go driver then found a run `paused` with no reason and wrote a friction note for it.

The two halts in #886 were one step: it skipped `Drift-Verify`, failed in `Code-Bouncer` on the git EAGAIN, and then its process died with exit status 2 before writing an envelope.
The loop blamed the row it had read before the child ran.
In #891 the stepped child died before its resume write over a goto-paused run, so the run stayed `paused` under its recorded reason, but the loop envelope did not carry that reason to the driver.

## What changes

- `internal/proc` gains `IsSpawnAgain`, the one test for an EAGAIN spawn refusal, and `NoteSpawnAgain`/`LastSpawnAgain`, a process-wide record of the last such refusal.
- `internal/gitexec`'s exec core starts git again when the start failed with EAGAIN: four tries, 250 ms then doubling, each retry logged at Warn and noted in the record.
  A git that ran is never started twice; a start still refused after the last try returns its exec error as before.
  `internal/reedengine`'s tmux retry uses the shared predicate and notes each refusal.
- `internal/shedengine` adds the transient class `host-resources`.
  `internal/shedtransient` returns it for an error carrying EAGAIN, and for an error wrapping the new `shuttleengine.ErrAgentDied` when this process noted a refusal within the last two minutes.
  The seats adapter wraps `ErrAgentDied` into a chair that started and then died.
- `internal/shedverbs` loop, for a child that ended without an envelope:
  - its cause carries its command line and a stderr excerpt, a Go crash's first lines (`panic:`, `fatal error:`, `runtime:`) when present and the last lines otherwise;
  - its trace file falls back to the captured stderr when the child left no trace;
  - the stop blames the row the status file names after the child exited;
  - the failed-stop write runs before `Hooks.AfterInterrupt`, so the hook reads the status the stop leaves.
- The loop object gains a `pause_reason` key, the status file's pause reason as the stop leaves it.
  `internal/sheddriver` takes it as the reason of a paused stop whose envelope has none, so such a stop no longer reads as a pause with no recorded reason.
- `internal/loomcli`'s interrupt hook records the status file's state in the halt note, which stays `paused` for a child that died before its resume write.
  An error in the status file that the stop did not write, such as the child's own producer failure, goes on a `status error:` line of the reason, which links the two halts of one step.
  `loomengine.ClassifyHalt` reports `interrupted-stop` for an interrupted halt that leaves the run failed or paused.
- Docs: the `doc.go` files of `proc`, `gitexec`, `reedengine`, `shedtransient`, `shedadapters`, `shedverbs`, `sheddriver` and `loomcli`.

## Design decisions to know

- A died agent counts as transient only with an EAGAIN refusal noted in the same step process within two minutes.
  A death with no refusal behind it and every timeout stay hard failures.
  The loop re-steps a transient once, and the driver hands one that outlives its re-step to the tackler as before.
- A child that died before its resume write leaves a paused run paused: the failed write still only writes into a file that reads running, so a deliberate pause is never overwritten.
  The stop's error still reaches the envelope and the halt note.
- The halt note's format is unchanged: the captured stderr goes in the existing `trace_file:` field, and the cause and the `status error:` line go in the reason.
- No change to lym's own spawn count at the end of Darn-Write.
  The exhaustion behind these issues was the user slice's task limit, filled host-wide by leaked MSBuild reuse nodes, which board entry `gate-robustness` addresses.

## To check by hand

- Not run: a real EAGAIN from the host.
  The tests stub the git start and note the refusal by hand.
- Not run: a live seats chair that dies after a tmux EAGAIN, re-stepped by the loop as `host-resources`.
