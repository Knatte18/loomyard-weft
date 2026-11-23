---
round: 3
focus:
  - "D5 in `_lym/discussion/decision-record.md`: settle whether the Decision closes the window between the rebuild shipping the key move and the first start verb, in which `hubreconcile.Ensure` (stamp-gated, called only from start verbs) has not yet moved the hub base `loom.yaml`, tracked worktree files or Seed-Child-written local files. Check that a non-start loom verb of an in-flight run then resolves old keys such as `code_max_bounces: 2` to their new paths without halting, and that D5's Bound lines cover files the walk has not yet reached, not only files that gain an old key after it."
  - "D4's load refusal and D5's rationale: settle whether every premise about when the walk runs matches where `hubreconcile.Ensure` and `EnsureOrWarn` are actually called, and whether an acceptance criterion pins the in-flight, stale-stamp case."
---

The only open finding is the window before the first start verb on a new build.
