---
round: 1
ledger:
  - key: test-only-functions-in-production-files
    rounds: [1]
    status: open
    class: decision
    severity: BLOCKING
    fix: design-wrong
  - key: reread-transport-failure-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: mixed-joined-read-error-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: multi-store-partial-fill-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: records-store-field-half-tested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: fabric-test-code-barely-shrinks
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: deleted-tests-without-loosened-check-trailer
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: layout-comments-name-engine-internals
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: smaller-loop-and-reader-branches-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 is the first round, so every key is new and open.

- One BLOCKING item: functions only tests reach sit in production files.
  The PATTERN entry forbids it and an added decision-record entry allows it, so the ruling is the parent's.
- Test gaps: the rule loop's error handling, the snapshot reader's multi-store and records-store reads, and smaller branches.
- The net shrink of fabric's test code is also for the parent to rule.
- Remaining: missing `Loosened-Check:` trailers and layout comments that name engine internals.
