---
round: 1
focus:
  - "internal/fabriccli/roleguard.go `inOwnSandboxRoot`: settle whether a path outside the worktree's own sandbox root can pass, through `..` segments, symlinks, a nested `.lym/harden-sandbox` inside a sandbox, or a lookalike directory."
  - "internal/fabriccli/roleguard.go `inOwnSandboxRoot`: settle whether the `ResolveWorktree` check (slug and worktree path) is correct for a worktree whose root is itself reached through a symlink, and for a failed resolve."
  - "internal/fabriccli/roleguard.go `verbTargetDir`: settle whether the directory the guard judges is the one the verb acts in for every guarded verb, including verbs that take a path or hub argument or a flag other than clone's `--into`, and the hidden teardown child of a removal."
  - "internal/fabriccli/clone.go `cloneDestination`: settle whether the extraction leaves `runCloneWithReset` behaving as before for empty, absolute and relative `--into`, and whether the guard and the clone resolve the same destination."
  - "internal/fabriccli/roleguard.go `sandboxedRoleRefusal`: settle whether the new pass sits where it cannot widen the merge-task and bypass-flag branches above it, and whether the changed refusal text still tells a sandboxed role the way forward."
  - "internal/fabriccli/roleguard_test.go `TestSandboxedRoleGuard`: settle whether its cases cover the refusals the commit message claims (real hub, another worktree's sandbox, lookalike directory) with literal expected values."
  - "Scope against the answer key (the Goal in the decision record): settle whether the stencil line in contracts/stencils/harden/harden-rubric-review.md and the doc.go addition are licensed, and whether the commit's Root-Cause trailer names the defect the diff removes."
---

Seed focus from the decision record's Goal and the committed diff of `a042662..HEAD`.
