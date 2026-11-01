---
round: 2
ledger:
  - key: placeholders-are-worktree-roots-not-anchors
    rounds: [1]
    status: resolved
  - key: env-removal-false-premise-unbounded-direct-call
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: nonzero-exit-opens-view-mode
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: mistyped-buttons-group-fails-config-loads
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: beam-placeholder-absent-or-landing-parent
    rounds: [1]
    status: resolved
  - key: cancel-task-session-cwd-blocks-removal-windows
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: percent-strftime-expanded-in-label-and-confirm
    rounds: [1]
    status: resolved
  - key: d3-d4-disagree-on-failed-desktop-load
    rounds: [1]
    status: resolved
  - key: lazy-beam-vs-closure-contract
    rounds: [1]
    status: resolved
  - key: bound-names-too-few-button-config-files
    rounds: [1]
    status: resolved
  - key: d1-bound-covers-only-booted-sessions
    rounds: [1]
    status: resolved
  - key: opus-high-unquoted-glob
    rounds: [1]
    status: resolved
  - key: task-command-anchor-run-dir-not-in-auto-assumptions
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised twelve findings, two BLOCKING.
Round 2 no longer reports the two BLOCKING ones: the worktree-root placeholder problem and the beam placeholder for an absent or landing parent are settled in the review's focus settlements, and the remaining round-1 LOW and NIT keys are not reported again, so they are resolved.

Four round-1 keys reappear in round 2, each as a narrower residue of its round-1 fix:

- `mistyped-buttons-group-fails-config-loads`: the fix claims a mapping-shaped group fails the load, but only desktop's machine layer refuses it (MEDIUM, gating).
- `nonzero-exit-opens-view-mode`: the forced zero status must sit in the pinned shell line, not only in the verb (MEDIUM, gating).
- `cancel-task-session-cwd-blocks-removal-windows`: the Windows rationale rests on a false premise about CANCEL (LOW).
- `env-removal-false-premise-unbounded-direct-call`: the identity claim fails on the confirm path (LOW).

One new key, `task-command-anchor-run-dir-not-in-auto-assumptions`, asks that running task commands in the beam's anchor be recorded under Auto-mode assumptions (MEDIUM consistency).
