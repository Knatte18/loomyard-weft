---
round: 2
ledger:
  - key: widest-stamped-line-omits-repository
    rounds: [1]
    status: resolved
  - key: sending-repository-replay-carry-unpinned
    rounds: [1]
    status: resolved
  - key: sending-repository-envelope-untested
    rounds: [1]
    status: resolved
  - key: reinterpretation-wiring-untested
    rounds: [1]
    status: resolved
  - key: transient-resolve-failure-drops-trust-entry
    rounds: [1]
    status: resolved
  - key: repository-segments-unchecked-before-stamp
    rounds: [1]
    status: resolved
  - key: summary-contradicts-resolve-error-handling
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised six `design` findings.
Round 2 found none of them again, and its focus verdicts cleared each area, so all six keys are resolved.
Round 2 raised one new finding, F1, mapped to `summary-contradicts-resolve-error-handling`:
the change description misstates which resolve errors drop a trust entry and lists a stale hub test row.
Its fix is a local text correction the finding already spells out.
