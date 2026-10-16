---
round: 2
ledger:
  - key: redundant-webster-wire-verify-tiers-row
    rounds: [1]
    status: resolved
  - key: unreachable-absent-verify-tiers-branch-in-load
    rounds: [1]
    status: resolved
  - key: listed-wire-row-never-carries-tiers
    rounds: [1]
    status: resolved
  - key: no-route-for-two-tier-render-rows
    rounds: [1]
    status: resolved
  - key: stale-comment-on-loom-wire-load-call
    rounds: [1]
    status: resolved
  - key: card4-test-helper-resign-missing-wrappers
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: tier-skip-bound-unstated-in-intents
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: rubric-marker-allowlist-doc-comment-stale
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised five findings, including one BLOCKING and one design-class MEDIUM.
Round 2 does not re-raise any of them, so all five are resolved.
Round 2 raised three new findings: F1 (re-sign arrows and wrappers on Card 4's test helpers), F2 (the tier-skip bound missing from the Intents of Cards 2 to 4) and F3 (a stale doc comment on `rubricMarkerAllowlist`).
Each fix is a local text correction the finding already specifies.
