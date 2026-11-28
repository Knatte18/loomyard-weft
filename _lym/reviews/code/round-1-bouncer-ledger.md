---
round: 1
ledger:
  - key: merge-danger-understates-prune-deletion-and-callers
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: worktree-gone-errors-on-file-at-path
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: parent-seams-pairof-directory-only-presence
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: worktree-gone-field-doc-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: summary-git-entry-rationale-false-premise
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings, none gating.
They cover the change description (merge danger, and the reason given for reading the `.git` entry),
how `worktreeGone` reads a regular file at the worktree path, and two places that still described or judged a pair
by its directory alone.
