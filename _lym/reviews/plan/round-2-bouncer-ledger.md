---
round: 2
ledger:
  - key: doc-go-nongo-file-targets
    rounds: [1]
    status: resolved
  - key: test-tier-unstated-way-forward
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: cards-8-9-redundant-rows
    rounds: [1]
    status: resolved
  - key: card4-new-test-file-via-handles
    rounds: [1]
    status: resolved
  - key: card8-test-pseudo-unit-row
    rounds: [1]
    status: resolved
  - key: card8-seam-reader-halt-no-actor
    rounds: [1]
    status: resolved
  - key: card6-external-struct-reflection
    rounds: [1]
    status: resolved
  - key: card4-tracked-root-head-narrowing
    rounds: [1]
    status: resolved
  - key: card4-trailing-paren-args
    rounds: [1]
    status: resolved
  - key: resolvetargets-doc-consumer-list
    rounds: [1]
    status: resolved
  - key: wirelightweight-doc-planindex
    rounds: [1]
    status: resolved
  - key: resolveglyphs-empty-list-unreachable
    rounds: [1]
    status: resolved
  - key: card9-validateplancmd-extends-vs-edit
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised twelve findings, none BLOCKING.
Round 2 raised two: F1, a MEDIUM design finding that reopens `test-tier-unstated-way-forward`, and F2, a LOW consistency finding on card 9's `validatePlanCmd` ref.
The other eleven round 1 keys did not recur and are resolved.
F1's fix extends a refusal's way-forward text, so it is `design-wrong` and needs a fresh review; F2's move from `Extends` to `Edit` is mechanical.
