```yaml
task: ubuntu-2404-support
filed:
  - https://github.com/Knatte18/loomyard/issues/758
  - https://github.com/Knatte18/loomyard/issues/759
dropped:
  - webster-refusal-record-batch.md
```

- #758: `loom-halt.md`, reed did not track the strand on liveness checks and the run failed.
- #759: `go-driver-paused.md`, run found paused with no reason (also cites `loom-halt.md`).
- `webster-refusal-record-batch.md` was dropped: the refusal is the verb working as designed, because a card's create target was not yet done, which is the task's own work.
