---
round: 2
ledger:
  - key: fast-forward-weft-bypasses-anchor-stops
    rounds: [1]
    status: resolved
  - key: owned-elsewhere-way-forward-loops
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: apply-failure-after-commit-no-end-state
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: anchor-validation-refusals-no-way-forward
    rounds: [1]
    status: resolved
  - key: unreachable-invalid-marker-teardown-fallbacks
    rounds: [1]
    status: resolved
  - key: fabricengine-write-allowlist-uncovered
    rounds: [1]
    status: resolved
  - key: drop-local-misses-unpushed-reanchor
    rounds: [1]
    status: resolved
  - key: create-at-anchor-engine-path-open
    rounds: [1]
    status: resolved
  - key: recovery-deletions-bypass-destroy-chokepoint
    rounds: [1]
    status: resolved
  - key: owner-refusal-exemption-unlicensed
    rounds: [1]
    status: resolved
  - key: create-record-removal-untimed
    rounds: [1]
    status: resolved
  - key: stale-docs-marker-on-weft-main
    rounds: [1]
    status: resolved
  - key: checkout-standing-record-refusal-unlicensed
    rounds: [1]
    status: resolved
  - key: integrate-weft-reset-keep-outside-destroy
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: move-record-removal-unnamed
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised thirteen findings; round 2 re-raised two of those keys and added two new ones.
The owned-elsewhere loop is back at BLOCKING: the unfinished-run refusal names a cancel the owner-elsewhere check refuses, and the plan needs a status read from the weft's upstream tip.
The post-commit failure end state is back for the errors between `BeginAnchorChange` and `Apply`.
New: `integrateWeft` resets the weft branch outside `destroy.go` (BLOCKING), and no card names the move record's remover (MEDIUM, scope).
Round 1's other keys were not re-raised and are carried as resolved.
