---
round: 1
ledger:
    - key: own-share-misses-call-processes-outside-guarded-tree
      rounds:
        - 1
      status: resolved
      class: design
      severity: BLOCKING
      fix: design-wrong
    - key: package-level-failure-ignored-by-slice-pressure
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: verify-sites-ignore-memory-pressure-stop
      rounds:
        - 1
      status: open
      class: scope
      severity: MEDIUM
      fix: design-wrong
    - key: pressure-closing-line-and-stops-call-untested
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: merge-copies-slice-pressure-untested
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: mixed-stops-and-failing-test-condition-untested
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: named-test-memory-use-not-measured
      rounds:
        - 1
      status: open
      class: scope
      severity: MEDIUM
      fix: mechanical
    - key: pressure-stop-unmarked-on-run-records
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
    - key: unreadable-holder-fallback-name-unpinned
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: within-cgroup-root-and-prefix-cases-unpinned
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
---
Round 1 opened every key above; there is no earlier history.
The open gating work is the own-share computation, the slice-pressure classification beside package-level failures,
and missing tests for the new pressure-stop behaviour.
The verify sites' handling of a pressure stop and the measurement the board entry asks for are open as scope items.
