```yaml
task: bugfix-sessions
notes_read:
  - loom-halt.md
  - webster-refusal-rebaseline.md
  - webster-refusal-record-batch.md
issues_filed:
  - https://github.com/Knatte18/loomyard/issues/413
```

All three notes describe one problem, so one issue was filed.
A loop-back to `Plan-Write` after batch 1 was begun left the plan drifted from Webster's recorded state.
`record-batch` advised `rebaseline`, and `rebaseline` refused, so the run halted `blocked`.
