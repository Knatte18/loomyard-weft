---
round: 1
ledger:
  - key: relay-never-ends-after-retire-drops-last-button
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: forwarding-entry-without-channel-entry-never-dropped
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: root-project-handle-outside-limits
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: worktree-orch-and-orch-project-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: failed-button-drop-after-archive-not-retried
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: pattern-daemon-wakeups-names-presence-watch
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: diskcheck-forwarding-comparison-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: hook-beam-orch-own-button-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: inbound-unreadable-button-branch-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised nine findings, none carried from an earlier round.
F1 (relay lifecycle), F2 (orphaned forwarding entry) and F4 (untested wiring) are gating.
F5 shares its pass with F2.
F3 is deferred to the run's parent.
