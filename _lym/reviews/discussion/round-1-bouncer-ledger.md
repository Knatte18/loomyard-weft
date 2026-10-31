---
round: 1
ledger:
  - key: record-write-body-headings-break-sections
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d10-half-finished-guard-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d13-retired-key-refusal-mechanism
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d4-inflight-darn-halt-unreachable
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d6-no-commit-halt-placement
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d6-noparent-answer-not-in-record
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d4-record-write-skip-not-resume-safe
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d12-plan-perch-hardcap-check-lost
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: scope-in-omits-pinned-specs
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: pattern-stop-chain-text-drops-batten-override
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d13-pawl-path-vs-scope-out
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d13-keycopies-migrateshedoverrides-unaddressed
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d3-peers-fallback-misreads-pawl
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d10-tapestry-preflight-map-skips-batcher-check
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d2-nonloom-refusal-placement-shared-arm
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: open-risk-batten-yaml-settled
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first review of the discussion pair, so the ledger starts here with every finding open.

The BLOCKING finding is that Record-Write's verbatim board body can inject H2 headings the record parser acts on.
The MEDIUM design findings share one pattern: a decision removes or moves an existing guard or path (D4, D6, D10, D12, D13) without stating the code mechanism that still holds the bound.
The scope, consistency and LOW findings are local corrections to the record's text.
