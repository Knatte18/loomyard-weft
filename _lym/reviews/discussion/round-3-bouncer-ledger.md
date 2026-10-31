---
round: 3
ledger:
  - key: record-write-body-headings-break-sections
    rounds: [1, 2]
    status: resolved
  - key: d10-half-finished-guard-unbounded
    rounds: [1]
    status: resolved
  - key: d13-retired-key-refusal-mechanism
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d4-inflight-darn-halt-unreachable
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d6-no-commit-halt-placement
    rounds: [1]
    status: resolved
  - key: d6-noparent-answer-not-in-record
    rounds: [1, 2]
    status: resolved
  - key: d4-record-write-skip-not-resume-safe
    rounds: [1, 2, 3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
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
    rounds: [2, 3]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d5-validate-guard-reads-fenced-headings
    rounds: [2, 3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d13-hub-tuned-retired-keys-dropped
    rounds: [2]
    status: resolved
  - key: d4-drift-baseline-resume-without-goto
    rounds: [2]
    status: resolved
  - key: d10-dropped-checks-no-acceptance-row
    rounds: [2]
    status: resolved
  - key: d4-differs-from-committed-restore-loses-answer
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d5-darn-guard-damaged-record-no-recovery
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
---

Round 3 closes the Record-Write body-heading BLOCKING key, the no-parent answer key, and the round 2 keys on hub-tuned retired keys, the Drift-Baseline resume and the D10 acceptance row.

Two gating keys recur.
The in-flight route to Darn-Write has been open in every round: round 3's route is refused by Shed's goto, and the single route `goto --to Record-Write` then `resume` replaces it.
The D13 retired-key check keeps landing in the wrong place; the local layer is judged by configengine, which is where the way-forward text belongs.

Two new `design` keys are raised in round 3: the restore-committed-copy way forward that discards an appended answer, and the Darn-Write guard's missing behaviour for a damaged record.
The remaining open keys are `consistency` and local: the darn refusal text by run row, the atomic-write helper in D4, and the fence-rule claim.
