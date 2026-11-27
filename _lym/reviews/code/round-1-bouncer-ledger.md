---
round: 1
ledger:
  - key: bench-teardown-refuses-held-run-lock
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: bench-removal-under-prime-lock
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: guard-admits-other-run-bench-slug
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: merge-danger-door-and-blast-radius
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: teardown-wiring-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: half-removed-bench-pair-skipped-silently
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: unreadable-run-record-misleading-refusal
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: guard-refusal-rows-missing
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: bench-removal-partial-failure-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: push-admission-skips-recipe-profile-check
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: pair-remote-pattern-omits-bench-route
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first round, and every key is open.
The findings fall in three areas:

- The bench pair teardown in `internal/pairteardown` and its Worktree-Teardown wiring: held run lock, prime-lock hold, silent skips, missing tests and the PATTERN text.
- The fabric role guard's admission of bench pairs: slug ownership, record read errors, push bound and missing rows.
- The change description's merge-danger call.
