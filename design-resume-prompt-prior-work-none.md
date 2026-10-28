# lym loom resume re-spawn prompt names prior work as (none) although the branch holds the first spawn's commit

`resume-prompt-prior-work-none` · Note · bug, loom, shed

Imported from [issue #889](https://github.com/Knatte18/loomyard/issues/889).

## What happened

After a run halted `blocked` at a producer, `lym loom resume` re-spawned the same producer.
The re-spawn's prompt named the prior work as `(none)`, while the branch already held the first spawn's commit and an archived summary.

## Why it is abnormal

The resumed agent is told there is no earlier work although there is, so it can redo or conflict with existing commits, or miss that the failure it must fix is a gate failure on already-written code.

## What to do

Have the resume path populate the prior-work section of the re-spawn prompt from the branch's existing commits and the archived summary of the earlier spawn.

Provenance: task `regression-harness`; note Darn-Write-2.md.
