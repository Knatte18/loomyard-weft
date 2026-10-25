---
round: 3
ledger:
  - key: fast-forward-weft-bypasses-anchor-stops
    rounds: [1]
    status: resolved
  - key: owned-elsewhere-way-forward-loops
    rounds: [1, 2]
    status: resolved
  - key: apply-failure-after-commit-no-end-state
    rounds: [1, 2]
    status: resolved
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
    status: resolved
  - key: move-record-removal-unnamed
    rounds: [2]
    status: resolved
  - key: card14-temp-index-no-gitexec-seam
    rounds: [3]
    status: open
    class: scope
    severity: MEDIUM
    fix: design-wrong
  - key: card8-bolt-marker-removal-unnamed
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card17-failure-branches-unreached
    rounds: [3]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: card17-owner-refusal-adopt-garbled
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card13-remove-move-record-callers-omit-close
    rounds: [3]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Rounds 1 and 2 raised design findings on the anchor-change flow: the owned-elsewhere loop, the post-commit end state and the destroy chokepoint.
Round 3 did not re-raise any of them, so all earlier keys are carried as resolved.
Round 3 found five new, non-gating items: card 14's temporary-index seam (MEDIUM, scope), and four LOW or NIT text corrections in cards 8, 13 and 17.
