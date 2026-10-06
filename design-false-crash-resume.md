# loom flags a clean producer handoff as a suspected driver crash

`false-crash-resume` · Note · bug, loom, shed

Imported from [issue #396](https://github.com/Knatte18/loomyard/issues/396).

## What happened

`lyx shed step` (the `step` verb) wrote a `loom-crash-resume` note ("loom resumed after a suspected driver crash at Webster-Bouncer") with `state: running`, `current_producer: Webster-Bouncer`.
The run history's last entry was `Webster / done`, recorded about six seconds before the step started.
The step then ran normally: a bouncer-seed shuttle run finished with `outcome=done`, and `shed: step done` reported `outcome=stuck next=Webster-Burler`.
Nothing in the trace shows a crash; the run was advancing from one producer to the next.

## Why it is abnormal

A crash-resume note is meant to mark a run that was last stopped without a clean handoff.
Here the previous producer completed and the next one had not yet started, so the note is most likely a false positive from the gap between two producers (the next row marked current, its run not yet begun).
The note also gives no reason for the "suspected" crash, so the operator cannot tell whether it is real.

## What to do

Check what `lyx shed step` / `lyx loom step` uses to decide that the previous driver crashed, and whether a run that is `running` with a freshly completed previous row should count.
Include the evidence (heartbeat, pid or lock state) in the note so a real crash and a false positive can be told apart.

Provenance: task `test-suite-measure`; notes: `loom-crash-resume.md`.
