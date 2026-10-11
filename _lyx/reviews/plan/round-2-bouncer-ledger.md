---
round: 2
ledger:
  - key: journal-scratch-dir-never-created
    rounds: [1]
    status: resolved
  - key: restore-card-name-rule-stricter-than-format
    rounds: [1]
    status: resolved
  - key: prose-mention-plural-list-not-reported
    rounds: [1]
    status: resolved
  - key: pattern-sole-parsers-write-path-list-stale
    rounds: [1]
    status: resolved
  - key: card4-restore-renumber-bound-misstated
    rounds: [1]
    status: resolved
  - key: archive-untouched-criterion-untested
    rounds: [1]
    status: resolved
  - key: card4-tests-duplicate-cards-2-3
    rounds: [1]
    status: resolved
  - key: non-integer-n-read-as-run-id
    rounds: [1]
    status: resolved
  - key: lease-test-scheduling-dependent
    rounds: [1]
    status: resolved
  - key: precheck-readable-journal-only
    rounds: [1]
    status: resolved
  - key: lease-test-release-unobservable
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: verbs-bounds-omit-writes-outside-plan-dir
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: reversal-not-yet-created-names-unstated
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: restore-journal-other-read-error-unmapped
    rounds: [2]
    status: open
    class: scope
    severity: LOW
---

Round 1 raised ten findings, including one BLOCKING; none was raised again in round 2, so all ten are resolved.
Round 2 raised four new findings, F1 through F4 in order.
F1 (lease test cannot observe a release and does not pin the lease start) is the one gating finding, MEDIUM design, first raised in round 2.
F2 through F4 are LOW precision and coverage gaps in the verbs' stated bounds, the reversal of not-yet-created names, and restore's unmapped read error.
