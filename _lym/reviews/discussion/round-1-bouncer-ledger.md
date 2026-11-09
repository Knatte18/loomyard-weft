---
round: 1
ledger:
  - key: verb-tests-write-real-machine-file
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: collision-check-shared-identity
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: repair-to-own-name-refused
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: hub-pause-advice-unfounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: owner-pause-ends-processes
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: sweep-misses-origin-only-owners
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: no-owner-for-file-path-and-reader
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: relative-xdg-config-home
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: slack-rename-back-unsafe
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: done-beams-keep-old-owner
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: lost-former-names-drop-landings
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: lift-before-sweep-skips-pair-pushes
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised twelve findings, all open; there is no earlier ledger.
The blocking one is that D8 isolates only `internal/machinename`'s tests, so the verb's tests would rename the developer's machine.
The MEDIUM findings concern the collision check, the repair path, the hub-pause advice, the owner-pause recovery, the sweep's reach and the missing owner for the file's path and reader.
The LOW findings are bounds and criteria to correct.
