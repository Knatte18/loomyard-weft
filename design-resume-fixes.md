# lym loom resume names prior work and honours a recorded wait decision

`resume-fixes` · Task · bug, loom, shed · running · Worktree-Create

## lym loom resume re-spawn prompt names prior work as (none) although the branch holds the first spawn's commit (was note `resume-prompt-prior-work-none`)

Imported from [issue #889](https://github.com/Knatte18/loomyard/issues/889).

#### What happened

After a run halted `blocked` at a producer, `lym loom resume` re-spawned the same producer.
The re-spawn's prompt named the prior work as `(none)`, while the branch already held the first spawn's commit and an archived summary.

#### Why it is abnormal

The resumed agent is told there is no earlier work although there is, so it can redo or conflict with existing commits, or miss that the failure it must fix is a gate failure on already-written code.

#### What to do

Have the resume path populate the prior-work section of the re-spawn prompt from the branch's existing commits and the archived summary of the earlier spawn.

Provenance: task `regression-harness`; note Darn-Write-2.md.

## lym loom resume re-prompts a producer to fix findings a recorded decision forbids, without checking the awaited task landed (was note `resume-honours-wait-decision`)

Imported from [issue #883](https://github.com/Knatte18/loomyard/issues/883).

A `lym loom` run halted blocked at a producer row after its verify gate failed three attempts in a row, and was resumed with `lym loom resume` before the condition the parent's recorded decision waited on had been met.

The decision told the run not to fix the failing findings and to wait for another task to land on `main`.
The resume re-spawned the producer with a fresh verify budget, and each round re-prompted it with "fix every finding".
That instruction contradicts the recorded decision, so every round could only end unchanged until the gate exhausted again, and the run halted blocked a second time with anomaly `escalation-to-human`.

This is abnormal because the resume burned a full verify budget and a producer session on a round known in advance not to converge.

What to do:
- Let `lym loom resume` (or the decision record) carry a "wait for task X to land" condition and refuse, or warn, while it is unmet.
- Or have the re-prompt after a resume include the recorded parent decisions, so "fix every finding" does not override them.

Provenance: task `readme-draft-not-ready`; notes Darn-Write.md, loom-halt.md, loom-halt-2.md.

