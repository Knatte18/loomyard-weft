---
round: 2
ledger:
  - key: record-write-body-headings-break-sections
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: d10-half-finished-guard-unbounded
    rounds: [1]
    status: resolved
  - key: d13-retired-key-refusal-mechanism
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d4-inflight-darn-halt-unreachable
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d6-no-commit-halt-placement
    rounds: [1]
    status: resolved
  - key: d6-noparent-answer-not-in-record
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d4-record-write-skip-not-resume-safe
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d12-plan-perch-hardcap-check-lost
    rounds: [1]
    status: resolved
  - key: scope-in-omits-pinned-specs
    rounds: [1]
    status: resolved
  - key: pattern-stop-chain-text-drops-batten-override
    rounds: [1]
    status: resolved
  - key: d13-pawl-path-vs-scope-out
    rounds: [1]
    status: resolved
  - key: d13-keycopies-migrateshedoverrides-unaddressed
    rounds: [1]
    status: resolved
  - key: d3-peers-fallback-misreads-pawl
    rounds: [1]
    status: resolved
  - key: d10-tapestry-preflight-map-skips-batcher-check
    rounds: [1]
    status: resolved
  - key: d2-nonloom-refusal-placement-shared-arm
    rounds: [1]
    status: resolved
  - key: open-risk-batten-yaml-settled
    rounds: [1]
    status: resolved
  - key: d4-decision-add-darn-refusals-way-forward
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d5-validate-guard-reads-fenced-headings
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d13-hub-tuned-retired-keys-dropped
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d4-drift-baseline-resume-without-goto
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d10-dropped-checks-no-acceptance-row
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 2 reopens five round 1 keys and raises five new ones.
Ten round 1 keys are resolved: round 2 does not raise them again.

The BLOCKING key is the Record-Write fence: round 1's fix chose a fence one longer than the longest backtick run, with no floor of three, so bodies with inline code only or no backticks still break the record's sections.
The reopened MEDIUM keys are the torn-write stall (D4), the retired-key check ordering against Migrate (D13), the no-parent answer path (D6), and the false re-capture claim on the in-flight no-commit route (D4).
The new MEDIUM keys are the `decision add` darn refusals that still name the Darn-Write session, the validate-only guard reading fenced headings, and hub-tuned retired keys dropped silently.
The two new LOW keys are local: the Drift-Baseline resume without the goto, and a missing D10 acceptance row.
