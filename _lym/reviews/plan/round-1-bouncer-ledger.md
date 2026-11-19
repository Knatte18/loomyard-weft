---
round: 1
ledger:
  - key: card8-pattern-line-narrows-rule
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: hardenshed-fabric-side-tokens
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: hardencli-package-naming-deviation-unlisted
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: hub-gate-key-values-dropped-unrecorded
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: liverd-absent-report-reads-green
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: refused-live-call-reads-as-failed-run
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: gateslot-doc-and-test-speed-tables-stale
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: new-test-packages-lack-testmain
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: preflight-multi-tier-copies-key
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: reap-orphan-kill-no-caller-spare
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: efforts-refusal-no-way-forward
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: mergeresolve-treadle-comment-missed
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: runproc-doc-names-one-endprocess-user
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: config-layers-strict-callers-omit-hardenshed
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: board-recipe-enumerations-omit-hardener
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first review, so every key is new and open.
Keys map to findings in review order:
F1 to F15 are `card8-pattern-line-narrows-rule`, `hardenshed-fabric-side-tokens`, `hardencli-package-naming-deviation-unlisted`,
`hub-gate-key-values-dropped-unrecorded`, `liverd-absent-report-reads-green`, `refused-live-call-reads-as-failed-run`,
`gateslot-doc-and-test-speed-tables-stale`, `new-test-packages-lack-testmain`, `preflight-multi-tier-copies-key`,
`reap-orphan-kill-no-caller-spare`, `efforts-refusal-no-way-forward`, `mergeresolve-treadle-comment-missed`,
`runproc-doc-names-one-endprocess-user`, `config-layers-strict-callers-omit-hardenshed` and `board-recipe-enumerations-omit-hardener`.
The three BLOCKING keys and the two design MEDIUM keys keep the loop open.
