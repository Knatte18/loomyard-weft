# Drift cannot be trialed in a sampler: every lym drift verb refuses outside drift stages

`sampler-drift-trial` · Note · feature, drift, shed

Imported from [issue #28](https://github.com/Knatte18/loomyard/issues/28).

## What happened

A `sampler` run in the Models hub (`drift-numerical-baseline`) had one job: trial the drift flow with the operator before drift is trusted on real tasks.
Every `lym drift` verb refused in the sampler worktree, because each verb is gated on the run's stage and the sampler recipe has no drift rows:

```
drift capture refused: the run is at Sampler-Session (running), and a main capture runs only at Drift-Baseline or at a Drift-Await halt
capture --side: the run is at Sampler-Session (running), and the verb runs only at Drift-Verify
compare: the run is at Sampler-Session (running), and the verb runs only at Drift-Verify
drift accept refused: the run is at Sampler-Session (running), and the verb runs only at Drift-Verify
withdraw: same as accept
```

The sampler fell back to running the hub's capture and compare commands by hand, and mimicked accept and withdraw with the skip file.
That proved the repository's harness, but none of lym's own drift stages: the Drift-Baseline agent's scope choice, `accept` and `withdraw` taking effect in the drift records, and the interactive Drift-Verify session.
The first real test of those stages is now a production task with drift full.

## Wanted

A way to trial drift in a sampler, the recipe meant for trying things with the operator.
For example, either:

- the drift verbs run in a sampler session, against a sampler-local drift record, so scope, capture, compare, accept and withdraw can be exercised end to end; or
- a sampler variant (or option) that carries the Drift-Baseline, Drift-Await and Drift-Verify rows, so the interactive Drift-Verify session can be reached with a deliberate change.

The stage gates stay as they are for tapestry and darn, where they keep an agent from re-baselining mid-task.