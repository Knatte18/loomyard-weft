---
round: 3
ledger:
  - key: plan-write-rotation-refusal-rows-missing
    rounds: [1]
    status: resolved
  - key: begin-batch-duplicate-table-test
    rounds: [1]
    status: resolved
  - key: reset-targets-test-misplaced-and-doubled
    rounds: [1]
    status: resolved
  - key: fingerprint-way-forward-callers-misstated
    rounds: [1]
    status: resolved
  - key: tokencount-startfit-write-breaks-build
    rounds: [1]
    status: resolved
  - key: standalone-reset-dirty-path-refusal-unspecified
    rounds: [1]
    status: resolved
  - key: refusal-spec-existing-rows-left-stale
    rounds: [1, 2, 3]
    status: open
    class: consistency
    severity: MEDIUM
  - key: reconcile-migration-duplicate-assertions
    rounds: [1]
    status: resolved
  - key: merriam-base-computed-on-recorded-partition
    rounds: [1]
    status: resolved
  - key: reset-keep-helper-in-wrong-card
    rounds: [1]
    status: resolved
  - key: reconcile-docs-not-in-card-targets
    rounds: [1]
    status: resolved
  - key: webster-yaml-refusal-untested
    rounds: [1]
    status: resolved
  - key: preflight-anchorpath-entry-table-row-missing
    rounds: [1]
    status: resolved
  - key: archive-only-path-runs-no-git-contradiction
    rounds: [1]
    status: resolved
  - key: standalone-keep-reset-violates-raw-git-guard
    rounds: [2]
    status: resolved
  - key: reset-to-start-bypasses-pending-findings-guard
    rounds: [2]
    status: resolved
  - key: plan-write-webster-deletions-unstaged
    rounds: [2]
    status: resolved
  - key: cards-miss-sites-a-grep-finds
    rounds: [1, 2, 3]
    status: open
    class: consistency
    severity: LOW
  - key: reset-start-card-repeats-keep-semantics
    rounds: [2]
    status: resolved
  - key: reset-targets-fakeable-refusals-in-integration-tier
    rounds: [2]
    status: resolved
  - key: plan-write-archive-collision-row-unreachable
    rounds: [2]
    status: resolved
  - key: computed-base-fixed-before-stencil-stops-changing
    rounds: [2]
    status: resolved
  - key: breakdown-base-has-no-reader
    rounds: [2]
    status: resolved
  - key: config-invalid-bound-vs-sentinel-reach
    rounds: [2]
    status: resolved
  - key: test-budget-rows-not-raised
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
  - key: reset-keep-drifts-pinned-run-bound-methods
    rounds: [3]
    status: open
    class: scope
    severity: BLOCKING
  - key: plan-write-seam-breaks-shared-fixture
    rounds: [3]
    status: open
    class: scope
    severity: MEDIUM
  - key: card-12-bundles-three-units
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: archive-test-repeats-existing-coverage
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
  - key: dirty-paths-no-seam-from-webstercli
    rounds: [3]
    status: open
    class: scope
    severity: LOW
---

Round 1 filed 14 findings and round 2 11; round 3 filed 8, two BLOCKING.
Round 3 confirmed the standalone keep-reset route, the pending-state handling, the refusal-spec row coverage and the Plan-Write rotation commit as acceptable, so those round-2 keys are resolved.
The round-2 LOW findings were not raised again and are resolved.

Round 3 key mapping follows the review's order:

- F1 maps to `test-budget-rows-not-raised`.
- F2 maps to `reset-keep-drifts-pinned-run-bound-methods`.
- F3 maps to `plan-write-seam-breaks-shared-fixture`.
- F4 maps to `refusal-spec-existing-rows-left-stale`; the stale row is now the `headBeforeEveryStart` refusal, not the earlier rows.
- F5 maps to `card-12-bundles-three-units`.
- F6 maps to `cards-miss-sites-a-grep-finds`; its class and severity are the latest review's (`consistency`, `LOW`).
- F7 maps to `archive-test-repeats-existing-coverage`.
- F8 maps to `dirty-paths-no-seam-from-webstercli`.
