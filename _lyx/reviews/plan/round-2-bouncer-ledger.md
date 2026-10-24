---
round: 2
ledger:
  - key: card-23-pattern-shorten-order
    rounds: [1]
    status: resolved
  - key: hand-enumerated-affected-sets
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: retag-cards-custom-not-edit
    rounds: [1]
    status: resolved
  - key: missing-resign-arrows-darntreescope-guardrunof
    rounds: [1]
    status: resolved
  - key: darnrouting-cannot-assert-entry-gate
    rounds: [1]
    status: resolved
  - key: entry-gate-error-points-at-run-settings
    rounds: [1]
    status: resolved
  - key: entry-gate-error-generic-resume
    rounds: [1]
    status: resolved
  - key: bare-config-reconcile-way-forward
    rounds: [1]
    status: resolved
  - key: card-24-verify-empty-covers
    rounds: [1]
    status: resolved
  - key: d13-follow-up-underspecified
    rounds: [1]
    status: resolved
  - key: default-gatekey-branch-test-only
    rounds: [1]
    status: resolved
  - key: greenrecord-duplicates-scenario-rows
    rounds: [1]
    status: resolved
  - key: retag-support-files-old-tag-spellings
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: card-20-duplicate-forced-answer-test
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card-21-extends-should-be-edit
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1's twelve findings are resolved except the affected-set method gap: round 2 does not reraise them and its Rejected section confirms the fixes hold.
`greenrecord-duplicates-scenario-rows` is not mentioned in round 2 and is treated as resolved on that silence.
The affected-set gap recurs as F1 (BLOCKING), now with five cards missing readers found by changed name or old output text.
Round 2 adds three new keys: support files naming old tag spellings (design, MEDIUM), a duplicate forced-answer test on card 20, and card 21's Extends-versus-Edit typing.
