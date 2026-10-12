# Loom halts a producer as failed with a bare 'watchdog' reason and no trace record of the kill

`watchdog-bare-reason` · Note · bug, loom, shed

Imported from [issue #566](https://github.com/Knatte18/loomyard/issues/566).

## What happened

A loom run halted `failed` at the `Plan-Write` producer with `anomaly: producer-hard-failure`.
The halt note's only reason text is the single word `watchdog`.
The trace of the final `shed step` ends five lines in, six seconds after the step started: `shed: step`, `claudeengine: resolved the prompt-cache TTL`, `shuttle: run started`, then one `shuttle: session state` line with `state=busy cause=turn`.
No later line records a watchdog firing, a timeout value, a kill, a stall or an error.
The halt note itself was written many hours after the last trace line.

## Why it is abnormal

The note and the trace together do not say which watchdog fired, what limit it enforced, how long the producer had been silent, or whether the shed process was killed or died on its own.
A reader cannot tell a genuine agent stall from a host suspend, a lost process or a watchdog misfire, so the halt cannot be triaged from the artefacts lyx leaves behind.

## What to do

- Have the watchdog path log a trace line when it fires: the watchdog's name, its configured limit, the elapsed silence and the action taken.
- Put that same detail in the halt note's reason instead of the bare `watchdog`.
- Check that a `shed step` process that ends without a clean exit leaves a final trace line saying how it ended.

Provenance: task `driver-arbiter`, note `loom-halt.md`.
