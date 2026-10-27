---
round: 1
ledger:
  - key: pre-run-exits-silent-in-dobby-log
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: concurrent-adopt-checks-share-record
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: adopt-check-before-migration
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: reused-pid-keeps-dead-check-running
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: reexec-test-by-pid-alone
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: tick-and-owner-read-lines-leave-dobby-log
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: rotation-has-two-writers
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: unreadable-machine-name-unbounded
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: fresh-dobby-unregistered-window
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: two-behaviours-lack-criterion
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: scope-out-bullet-wrong-entry
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: decision-4-reason-mismatch
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: migration-not-once
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: decision-8-bound-lacks-evidence
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: constraints-bullet-repeats
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised fifteen findings, none carried from an earlier round.
Three are MEDIUM design findings: silent pre-`Run` exits in `dobby.log`, two concurrent adopt checks sharing one record and log, and the first adopt check reading the template's `adopt` before the migration runs.
The LOW design findings cover pid reuse, the re-exec test, lost episode lines, rotation, an unreadable machine name and the unregistered window of a fresh dobby.
The rest are consistency items, mostly missing criteria and wording.
