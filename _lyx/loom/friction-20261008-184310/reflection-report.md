# Reflection report

Filed:

- https://github.com/Knatte18/loomyard/issues/467 — `Publish` failed in merge-in with "worktree status: object not found" (from `loom-halt-2.md`).

Dropped:

- `loom-halt.md`: a `Preflight` block on the `worktree-clean` precondition with uncommitted `_lyx` config changes.
  This is an escalation to the operator, a process signal rather than a lyx defect.
