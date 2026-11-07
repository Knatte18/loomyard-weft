---
round: 1
ledger:
  - key: run-invocations-test-targets-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: example-functions-dropped-from-test-targets
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: stale-targets-only-wording
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: test-targets-fallback-and-pattern-rows-missing
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: no-tier-adopts-test-targets
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings (F1 to F5), one gating.
F1 (MEDIUM design) is the missing test for the `{test_targets}` wiring in `runInvocations`.
F2 to F4 are LOW corrections: Example functions, stale `{targets}` wording and missing test rows.
F5 is a LOW decision finding deferred to the run's parent.
