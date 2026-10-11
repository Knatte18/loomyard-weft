---
round: 2
ledger:
  - key: card9-untargeted-recipe-and-registry-tests
    rounds: [1]
    status: resolved
  - key: land-pause-flag-stays-set-on-resume
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: unreached-stuck-and-refusal-branches
    rounds: [1, 2]
    status: open
    class: scope
    severity: LOW
  - key: orch-stencil-run-loop-contradicts-publish-retry
    rounds: [1]
    status: resolved
  - key: card11-targets-claim-struct-not-wordingclaims
    rounds: [1]
    status: resolved
  - key: push-retry-loop-tested-twice
    rounds: [1]
    status: resolved
  - key: loadconfig-deploy-per-key-test
    rounds: [1]
    status: resolved
  - key: batten-status-seam-tests-split-per-field
    rounds: [1]
    status: resolved
  - key: conditiontext-edit-unspecified
    rounds: [1]
    status: resolved
  - key: goto-teardown-bypasses-land-unstated
    rounds: [1]
    status: resolved
  - key: halted-over-done-child-text-stale
    rounds: [1]
    status: resolved
  - key: orchcli-doc-go-landing-routine-stale
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
  - key: retryable-push-refusal-needlessly-exported
    rounds: [1]
    status: resolved
  - key: push-primitive-retry-ordering-untested
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
  - key: ready-notice-routed-to-investigating-stop
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: row-notice-episode-fallback-untested
    rounds: [2]
    status: open
    class: scope
    severity: LOW
  - key: acting-without-asking-still-lists-land
    rounds: [2]
    status: open
    class: scope
    severity: LOW
  - key: after-landing-claim-count-wrong
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
  - key: landentry-file-unnamed
    rounds: [2]
    status: open
    class: scope
    severity: NIT
---

Round 1 raised 13 findings, one BLOCKING and three gating design.
Round 2 raised nine, none BLOCKING and none design, so the loop converged.
Three round 1 keys recur in a changed form: the Land pause test (F1), the unreached Stuck branches (F4) and the orchcli doc run loop (F6).
The remaining round 1 keys are resolved, and the other six round 2 findings are new keys.
