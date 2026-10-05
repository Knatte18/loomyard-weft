---
round: 1
ledger:
  - key: comment-hard-wrap-mid-sentence
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
  - key: new-tests-missing-t-parallel
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
  - key: commitwritten-uncovered-behaviors
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: configengine-doc-rewrap-note-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: reconcile-help-omits-hubwide-repair
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 is the first review; there is no earlier ledger.
Five findings are open: two BLOCKING comment and test-convention violations, one MEDIUM coverage gap for `Bolt.CommitWritten`, and two LOW documentation gaps.
None is `design` class, so the loop continues only because of the BLOCKING severity.
