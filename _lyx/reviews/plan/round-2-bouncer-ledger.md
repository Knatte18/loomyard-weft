---
round: 2
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
    rounds: [1, 2]
    status: open
    class: design
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
    status: open
    class: design
    severity: BLOCKING
  - key: reset-to-start-bypasses-pending-findings-guard
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: plan-write-webster-deletions-unstaged
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
  - key: cards-miss-sites-a-grep-finds
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: reset-start-card-repeats-keep-semantics
    rounds: [2]
    status: open
    class: design
    severity: LOW
  - key: reset-targets-fakeable-refusals-in-integration-tier
    rounds: [2]
    status: open
    class: design
    severity: LOW
  - key: plan-write-archive-collision-row-unreachable
    rounds: [2]
    status: open
    class: design
    severity: LOW
  - key: computed-base-fixed-before-stencil-stops-changing
    rounds: [2]
    status: open
    class: design
    severity: LOW
  - key: breakdown-base-has-no-reader
    rounds: [2]
    status: open
    class: scope
    severity: LOW
  - key: config-invalid-bound-vs-sentinel-reach
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1 filed 14 findings; round 2 found its F8 to F14 fixed and its F1 to F6 resolved.
Round 1 F7 recurs as F2 (stale refusal-spec rows in cards 11, 12 and 14).
Round 1 F4, F5, F11 and F13 (individually missed sites) recur as the single method finding F5, key `cards-miss-sites-a-grep-finds`.
Round 1 F10 (reset-keep helper placement) was fixed by moving the call into card 12, which regressed into the BLOCKING F1.
Round 1 F9 (merriam base on a recorded partition) is resolved; the base now has its own LOW finding about the derivation point.
Round 2 key mapping follows the review's order: F1 raw-git guard, F2 stale rows, F3 pending findings, F4 unstaged deletions, F5 method gap, F6 to F11 the LOW findings.
