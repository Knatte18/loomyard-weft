---
round: 1
ledger:
  - key: hold-setters-no-production-caller
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: handover-mark-drop-unbuilt
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: root-beam-heads-chain
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: adopt-single-bolt-commit-setter
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: fix-tries-reader-not-kept
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: launcher-removal-misses-catchup
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings, all open.
Two BLOCKING design findings concern contracts that land functions no production code calls: the hold setters and the hand-over mark drop.
Two MEDIUM design findings concern the chain head and the missing setter for D6 step 1's single Bolt commit.
Two LOW scope findings concern the try-count reader and the launcher removal at every catch-up.
The root-beam fix depends on where F1's fix puts `internal/boardengine`.
