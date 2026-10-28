---
round: 3
ledger:
  - key: doc-go-nongo-file-targets
    rounds: [1]
    status: resolved
  - key: test-tier-unstated-way-forward
    rounds: [1, 2, 3]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
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
    status: resolved
  - key: untagged-helper-beside-tagged-row
    rounds: [3]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: tag-set-or-count
    rounds: [3]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
  - key: validate-long-help-owner
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised twelve findings, round 2 two, round 3 four; none was ever BLOCKING.
The way-forward key `test-tier-unstated-way-forward` recurs in round 3 as a consistency finding: the string never offers the `untagged` form for a support-only file.
Its fix is a local rewording the finding text already fixes, so it is `mechanical`.
The other three round 3 findings are new, LOW, and each names its text change.
`card9-validateplancmd-extends-vs-edit` is resolved: round 3 confirms all three cards type the ref `Edit`.
