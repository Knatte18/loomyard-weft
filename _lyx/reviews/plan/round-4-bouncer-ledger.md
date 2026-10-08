---
round: 4
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
    status: resolved
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
    rounds: [1, 2, 3, 4]
    status: open
    class: consistency
    severity: NIT
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
    status: resolved
  - key: reset-keep-drifts-pinned-run-bound-methods
    rounds: [3]
    status: resolved
  - key: plan-write-seam-breaks-shared-fixture
    rounds: [3]
    status: resolved
  - key: card-12-bundles-three-units
    rounds: [3]
    status: resolved
  - key: archive-test-repeats-existing-coverage
    rounds: [3]
    status: resolved
  - key: dirty-paths-no-seam-from-webstercli
    rounds: [3]
    status: resolved
  - key: archive-only-pending-findings-dead-end
    rounds: [4]
    status: open
    class: design
    severity: BLOCKING
  - key: edit-bullets-lack-resign-arrow
    rounds: [4]
    status: open
    class: design
    severity: MEDIUM
  - key: edit-cards-list-files-not-members
    rounds: [4]
    status: open
    class: consistency
    severity: LOW
  - key: cards-13-22-bundle-units-uncarded-exports
    rounds: [4]
    status: open
    class: design
    severity: LOW
  - key: card-17-reimplements-uncommitted-paths
    rounds: [4]
    status: open
    class: consistency
    severity: LOW
  - key: keep-reset-refusal-test-duplicates-resetcmd-step
    rounds: [4]
    status: open
    class: design
    severity: LOW
---

Round 1 filed 14 findings, round 2 11, round 3 8 and round 4 7.
Round 4 found none of the round-3 findings again: the test-budget rows, the pinned run-bound methods, the shared fixture, the three-card split and the `UncommittedPaths` seam all held, so those keys are resolved.
`refusal-spec-existing-rows-left-stale` is resolved too; round 4 settles the changed-text sites.

Round 4 key mapping follows the review's order:

- F1 maps to `archive-only-pending-findings-dead-end`; it comes from the round-3 fix that named `accept-audit` as the reset caller's step.
- F2 maps to `edit-bullets-lack-resign-arrow`.
- F3 maps to `edit-cards-list-files-not-members`.
- F4 maps to `cards-13-22-bundle-units-uncarded-exports`.
- F5 maps to `card-17-reimplements-uncommitted-paths`.
- F6 maps to `keep-reset-refusal-test-duplicates-resetcmd-step`.
- F7 maps to `cards-miss-sites-a-grep-finds`; its class and severity are the latest review's (`consistency`, `NIT`).
