---
round: 1
ledger:
  - key: moved-checks-way-forward-cannot-clear-halt
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: non-uncaused-ruling-reuse-across-range-change-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: overview-omits-third-way-forward-and-rejudge
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: check-and-judge-comment-states-old-reuse-rule
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: rejudge-trigger-wording-overclaims
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: range-key-content-coverage-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: range-key-trailer-input-redundant
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 is the first round.

The central open point is that the diff re-judges a stored uncaused ruling once the range changes,
but the judge stencil's file rules still make the judge rule moved checks `uncaused` without a `Loosened-Check:` trailer.
The way forward the halt names therefore leads back to the halt.

The other open points are two test gaps on the reuse rule and the range key,
stale or overclaiming wording in the overview, comments, halt text and change description,
and a redundant input to the range key.
