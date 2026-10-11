---
round: 1
ledger:
  - key: card9-untargeted-recipe-and-registry-tests
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
  - key: land-pause-flag-stays-set-on-resume
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: unreached-stuck-and-refusal-branches
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: orch-stencil-run-loop-contradicts-publish-retry
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: card11-targets-claim-struct-not-wordingclaims
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: push-retry-loop-tested-twice
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: loadconfig-deploy-per-key-test
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: batten-status-seam-tests-split-per-field
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: conditiontext-edit-unspecified
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: goto-teardown-bypasses-land-unstated
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: halted-over-done-child-text-stale
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: orchcli-doc-go-landing-routine-stale
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: retryable-push-refusal-needlessly-exported
    rounds: [1]
    status: open
    class: design
    severity: LOW
---

Round 1 is the first review of a generation-0 plan, with 13 findings and none recurring.
The BLOCKING finding is card 9's missing targets for tests that pin the old four-row recipe and the closed registry.
The gating design findings are the pause flag surviving a blocked Land, unreached Stuck and refusal branches across cards 8 to 10, and the push retry loop tested in both card 1 and card 2.
