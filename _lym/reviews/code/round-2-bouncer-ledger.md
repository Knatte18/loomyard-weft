---
round: 2
ledger:
  - key: cancelled-step-kept-as-complete
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: stale-hash-cover-exclusion-unpinned
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: runtest-wiring-untested
    rounds: [1]
    status: resolved
  - key: lym-build-and-corrupt-map-untested
    rounds: [1]
    status: resolved
  - key: spawn-scan-drops-untested
    rounds: [1]
    status: resolved
  - key: mid-package-budget-untested
    rounds: [1]
    status: resolved
  - key: retry-error-line-and-match-false-cases-untested
    rounds: [1]
    status: resolved
  - key: failed-report-write-and-slot-hold-untested
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: cancelled-reason-untested
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised eight design findings; round 2 raised four.
Five round 1 keys are resolved: the wiring, the `lym` build and corrupt map, the spawn scan, the mid-package budget and the retry rows now have tests.
F1 (in-flight test killed by a cancelled call, MEDIUM) maps to `cancelled-step-kept-as-complete`.
F2 (no test produces `reasonCancelled`, LOW) is the new key `cancelled-reason-untested`.
F3 (judged-name exclusion unpinned, LOW) maps to `stale-hash-cover-exclusion-unpinned`.
F4 (slot hold not asserted, LOW) maps to `failed-report-write-and-slot-hold-untested`.
