# Stale batcher.yaml key startup_context is only caught at record-batch, halting a run mid-Webster

`config-drift-preflight` · Note · bug

Imported from [issue #440](https://github.com/Knatte18/loomyard/issues/440).

**What happened**
A loom run reached the Webster producer and halted `blocked` (anomaly `escalation-to-human`) when `lyx webster record-batch` refused with: `batcher.yaml did not load: profile "cautious" carries weights key startup_context, which no longer exists; set master_base and batch_growth under _lyx/config`.
The pair's `_lyx/config/batcher.yaml` still carried a key that the current binary no longer accepts.
The run had already passed Preflight, Loom-Preflight, Discussion, Plan and Batchifier, so the schema drift surfaced only after the Webster batch had done its work and a report was written.
The operator-side orchestrator had to hand-edit the pair's config and re-step the run once (`lyx shed step`), because the stop was a binary-changed stop.

**Why it is abnormal**
A config file that cannot load is deterministic and detectable up front.
Nothing in Preflight or Loom-Preflight validated `batcher.yaml`, so the run burned its whole Discussion/Plan phase before the failure.
The error also lands as a human escalation rather than a clear config-validation failure.

**What to do**
- Load and validate `batcher.yaml` (and the other per-pair config) in Preflight/Loom-Preflight so schema drift fails at the start.
- Consider migrating or ignoring removed keys such as `startup_context` when the pair config is seeded from an older template, or have the seeding step regenerate the file.

Provenance: task slug `lifecycle-waits`; notes `loom-halt.md`, `driver-restep-webster-20261007.md`.
