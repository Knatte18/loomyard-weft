---
round: 1
ledger:
  - key: stamp-builder-untested
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: test-file-import-miss
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: stamp-miss-rows-incomplete
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: build-failure-beside-tests-misses
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: unreadable-commit-target-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: unreadable-toolchain-miss-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: package-clause-change-untested
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 raised seven findings, all `design`: F1 maps to `stamp-builder-untested`, F2 to `test-file-import-miss`,
F3 to `stamp-miss-rows-incomplete`, F4 to `build-failure-beside-tests-misses`, F5 to `unreadable-commit-target-untested`,
F6 to `unreadable-toolchain-miss-untested`, and F7 to `package-clause-change-untested`.
No earlier ledger exists.
F2 is the one real logic change; the rest are missing tests, plus F4's one-line condition.
