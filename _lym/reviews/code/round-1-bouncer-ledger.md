---
round: 1
ledger:
  - key: unconverted-lym-steps-weftdirtystop-upstreamanchorstop
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: lymverbdir-calls-lack-anchored-tests
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: branchrewritten-dead-records-branch
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: lymverbdir-fallbacks-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: doc-claims-every-way-forward
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings, all open.
F1 (scope, MEDIUM): `weftDirtyStop` and `upstreamAnchorStop` still name the bare root.
F2 (design, MEDIUM): converted `lymVerbDir` lines lack anchored test rows; this gates the round.
F3 (design, LOW): dead records branch in the `BranchRewritten` way forward.
F4 (design, LOW): `lymVerbDir` fallbacks lack rows.
F5 (consistency, LOW): the `doc.go` sentence claims every way forward.
