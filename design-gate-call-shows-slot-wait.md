# Timing a gate test call includes the slot wait; run time needs the run record

`gate-call-shows-slot-wait` · Task · enhancement, gate · running · Darn-Write

Imported from [issue #284](https://github.com/Knatte18/loomyard/issues/284).

## What happened

A session timing packages through `lym gate test` found that a call's wall time includes the wait for a gate slot,
up to 300 s in that run.
Timing the calls therefore did not measure the test runs, and the session took its wall figures from `wall_seconds` in each run record.

## Why it is a problem

A caller who measures a gate call from outside gets a figure dominated by contention for slots,
and has to know about the run records to get the run's own time.
The note does not say whether the call's own output separates the two.

## What to do

Check that the result of `lym gate test` shows the slot wait and the run's wall time as separate values.
If it does not, add them; if it does, document in the verb's help where to read the run time without the wait.

Provenance: task `test-time-measure`, note `Darn-Write.md`.
