---
round: 2
ledger:
  - key: card2-hub-worktreeroot-is-anchor-path
    rounds: [1]
    status: resolved
  - key: card2-whitespace-skip-unbounded
    rounds: [1]
    status: resolved
  - key: stale-lyx-pattern-refs-uncovered
    rounds: [1]
    status: resolved
  - key: sweep-cards-list-noop-files
    rounds: [1, 2]
    status: open
  - key: card18-scan-boundaries-undecided
    rounds: [1]
    status: resolved
  - key: card20-spec-type-unnamed
    rounds: [1]
    status: resolved
---

Round 2 mapping: F1 maps to `sweep-cards-list-noop-files`.
It is LOW scope, so it is not gating, and it now names two other files (`reedgeom_integration_test.go`, `smoke_live_test.go`).
The round 2 review settled the repo-root, whitespace-skip and card 18 boundary questions without defect, so those three keys are resolved.
The review raised nothing on `stale-lyx-pattern-refs-uncovered` or `card20-spec-type-unnamed`, so both are resolved.
