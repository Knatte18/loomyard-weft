# Webster recovery ending dead with dead_reason asking drops the question and needs a manual reset

`webster-recovery-asking-dead` · Note · bug, webster

Imported from [issue #220](https://github.com/Knatte18/loomyard/issues/220).

## What happened

A Webster batch's verify failed on a test the plan could not satisfy.
The batch recovery session ended `dead` with `dead_reason: asking`, i.e. the recovery agent stopped to ask a question instead of finishing, and `recoveries: 1`, `recovery_retry: false`.
Then:

```
lym webster recover-batch 3 refused:
webster: batch 03-torque-drag-stage recovery (session ...) ended dead and the batch has no recovery left; way forward: end the run stuck, naming batch 03-torque-drag-stage
```

The run halted `blocked` (anomaly `escalation-to-human`).
After the plan was edited and rebaselined, Webster halted `blocked` again:

```
webster: a batch of this run is at a dead end: batch 03-torque-drag-stage is dead and its recovery earns no retry, so a Master would only end the run stuck on it; way forward: 1) lym webster reset --to batch-start --batch 03; 2) re-step the Webster row; or 1) lym webster reset --to start; 2) re-step the Webster row
```

## Why abnormal

A recovery agent that asks a question has no channel that answers it, so the question becomes a dead recovery with no retry.
The agent's question, which would tell the operator what is wrong, is not carried into the halt text: the halt note only restates "dead_reason: asking".
Plan edits made to fix the cause did not clear the dead state either; a manual `webster reset` was needed.

## What to do

- When a recovery ends with `dead_reason: asking`, put the agent's last message in the halt/refusal note.
- Either route the question to the parent (`lym creel post --parent --expect-reply`) or let a plan rebaseline that touches the dead batch's cards re-arm one recovery, so the operator does not need a reset.

Provenance: task `models-torque-drag-adapter`; notes `loom-halt.md`, `loom-halt-5.md`, `webster-refusal-recover-batch.md`.
