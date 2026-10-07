---
round: 2
ledger:
  - key: join-leaves-timed-out-half-live
    rounds: [1]
    status: resolved
  - key: recipe-comment-redundant-clause
    rounds: [1]
    status: resolved
  - key: burler-help-overlong-line
    rounds: [1]
    status: resolved
  - key: timed-out-half-failed-stop-untested
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
  - key: join-errored-half-left-unstopped
    rounds: [2]
    status: open
    class: design
    severity: LOW
  - key: join-log-call-gofmt
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
---

Round 1's three findings are resolved: a fresh round 2 review confirmed the timeout stop in `join` and judged the recipe comments and the burler help paragraph not a defect.
Round 2 raised three new findings.
F1 (scope, MEDIUM): no test row for a timed-out half whose stop fails.
F2 (design, LOW): `join` does not stop a half whose `Wait` returned an error.
F3 (consistency, NIT): a `logger.Info` call in `handoff.go` fails gofmt.
None is gating.
