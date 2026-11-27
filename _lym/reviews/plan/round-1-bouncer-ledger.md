---
round: 1
ledger:
  - key: fabric-push-bypasses-records-hold
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: beam-parent-push-ceiling-unpassable
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: ledger-removals-drop-landings
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: held-only-pair-never-level
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: unrecorded-landing-refusal-dead-way-forward
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: landing-state-only-in-lym-cache
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: tests-stand-in-for-repo-modules
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: pushledger-tests-lack-tmuxkit-main
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: archive-parent-merge-left-test-only
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: beam-pr-head-missing-on-origin
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: design-wrong
  - key: spawn-owner-test-row-breaks
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: dobbyengine-doc-stale
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: pairteardown-doc-stale
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: landing-records-commit-before-hold
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: spawn-criterion-tests-incomplete
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: pattern-shed-driver-background-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: cancel-intent-committed-before-landing-refusal
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: overview-applies-to-misplaced
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: hub-clear-time-unset-before-first-clear
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 is the first round, so every key is new and open.

The design findings cluster on the landing ledger and the records hold:
which push paths apply the hold, what ceiling a beam parent push gets, which removals may drop an entry,
how a held pair reads as level, what survives a lost ledger, and the order of the landing's commit and its hold.
The remaining findings are test rules, one function left without a production caller, stale docs and one overview line.
