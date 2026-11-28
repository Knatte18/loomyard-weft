# Darn writer prompt forbids the gate calls a measurement entry requires

`darn-writer-measure-gates` · Task · bug, darn, stencil · done

Imported from [issue #283](https://github.com/Knatte18/loomyard/issues/283).

## What happened

A darn writer session ran an entry that is itself a measurement: one `lym gate test --tier <n> <package>` call per package and tier.
The writer's gate rules in `contracts/stencils/darn/darn-fragment-make-change-gated.md` say to run no `lym gate test`,
bar one test class through the measuring call, and name two exceptions, neither of which covers measurement.

The writer followed the entry over the prompt: several hundred calls with `--wait --no-carry`, run in loops, about 70 minutes.
It did not ask its parent first.

## Why it is abnormal

The prompt and the entry contradict each other, and the prompt says nothing about which wins.
A writer that obeys the prompt cannot do the entry; one that does the entry breaks a stated rule on its own decision.

## What to do

Give the writer prompt a rule for an entry whose deliverable is gate measurements: either state that those gate calls are allowed
(and with what limits), or tell the writer to ask its parent before running them.

Provenance: task `test-time-measure`, note `Darn-Write.md`.
