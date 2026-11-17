---
round: 1
ledger:
  - key: shell-sigkill-reads-as-memory-cap-kill
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: guard-kill-in-runalone-reads-as-cap-kill
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: redundancy-report-step-unguarded
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: tier-run-never-names-killed-test
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: design-wrong
  - key: unsettable-memory-cap-no-cause-or-way-forward
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: tier-run-memory-kill-wiring-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: tiers-still-uncapped-no-default
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
  - key: guardowntests-start-stop-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: cpu-watchdog-sentence-misplaced-after-memory-section
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: stderr-field-comment-misses-new-lines
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised ten findings, none carried from an earlier round.
Four are gating design findings: how a kill is attributed (the tier shell's SIGKILL and `RunAlone`'s guard kill both read as a `memory_mb` kill), the unsettable-cap refusal, and the missing tier-run test.
Two MEDIUM scope findings concern the unguarded redundancy report step and the unnamed killed test.
The remaining four are LOW: the uncapped tiers (deferred to the parent), the `guardOwnTests` test, and two doc and comment fixes.
