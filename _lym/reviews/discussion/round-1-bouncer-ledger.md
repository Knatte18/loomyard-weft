---
round: 1
ledger:
  - key: open-uses-readonly-cwd-resolver
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: token-file-protection-windows
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: liveness-read-trusts-unwritten-or-draining-state
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: server-tied-to-opener-worktree
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: csp-location-and-page-mode-unstated
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: older-binary-replays-newer-sync-commits
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: release-running-task-no-bound
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: tool-facts-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: dropped-refusal-and-render-outputs-parent-call
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: orch-open-blocks-on-foreground-browser
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: criteria-miss-trust-check-and-refusals
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: write-revision-not-pinned-to-check
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: render-undiscussed-copy-vs-revision
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: server-runproc-role-unnamed
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: create-refusal-raddle-slug-no-entry
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: support-log-contradicts-d9-d12-bounds
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: notes-restate-d6-d8
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d3-last-bound-sentence-moot
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised 18 findings; all are open.
The two BLOCKING ones are `open-uses-readonly-cwd-resolver` and `token-file-protection-windows`.
The `liveness-read-trusts-unwritten-or-draining-state` and `server-tied-to-opener-worktree` findings both touch D11's server start and exit, so their fixes need review together.
`dropped-refusal-and-render-outputs-parent-call` goes to the run's parent, not the fixer.
`d3-last-bound-sentence-moot` depends on how `older-binary-replays-newer-sync-commits` is fixed.
