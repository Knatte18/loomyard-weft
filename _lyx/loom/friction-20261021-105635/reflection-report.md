```yaml
task: run-reentry
issues_filed: none
```

Nothing filed.

- `webster-refusal-begin-batch.md`: `begin-batch 12` refused with `plan_conflict: tree_moved` because a Create target already resolves in the tree.
  The refusal is the intended guard, and its message names the way forward (edit the plan, `lyx webster rebaseline`).
  The cause is the plan versus the task's own tree, not a lyx defect.
- `webster-refusal-record-batch.md`: `record-batch 01` refused because a Create target did not resolve after the batch.
  The batch's cards were not completed, and the refusal names `lyx webster recover-batch 01`.
  The dirty-worktree warning is a plain consequence of the failed batch; the notes show no lyx fault behind it.
