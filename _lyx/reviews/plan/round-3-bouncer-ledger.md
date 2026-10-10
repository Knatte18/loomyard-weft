---
round: 3
ledger:
  - key: card11-registry-names-pin-missing
    rounds: [1]
    status: resolved
  - key: card6-done-advisor-notifies-chair
    rounds: [1]
    status: resolved
  - key: card6-misnamed-seat-strand-left-running
    rounds: [1]
    status: resolved
  - key: cards3-5-seat-channel-bound-unstated
    rounds: [1]
    status: resolved
  - key: card5-reserved-marker-refusal-unbuilt
    rounds: [1]
    status: resolved
  - key: card3-chair-block-channel-wording
    rounds: [1]
    status: resolved
  - key: card6-chair-wait-error-unspecified
    rounds: [1]
    status: resolved
  - key: card11-retests-unknown-row-key
    rounds: [1]
    status: resolved
  - key: card12-repeats-nested-include-refusal
    rounds: [1]
    status: resolved
  - key: card5-uses-embedded-seat-block-vars
    rounds: [1]
    status: resolved
  - key: reserved-markers-split-cards-4-5
    rounds: [1]
    status: resolved
  - key: card10-envkit-fullenv-battencli-seams-missing
    rounds: [2]
    status: resolved
  - key: card11-shedbuild-every-engine-build-test-missing
    rounds: [2]
    status: resolved
  - key: card2-3-template-include-passes-no-data
    rounds: [2]
    status: resolved
  - key: card6-9-n1-equivalence-pinned-only-via-fake
    rounds: [2, 3]
    status: open
    class: design
    severity: LOW
  - key: card2-include-pipeline-dot-not-root
    rounds: [3]
    status: open
    class: design
    severity: LOW
  - key: card2-includenames-leading-comment-unstripped
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
  - key: card1-send-lock-unspent-attempt-unasserted
    rounds: [3]
    status: open
    class: scope
    severity: LOW
---

Rounds 1 and 2 raised fifteen findings, none gating at round 1 and the BLOCKING and MEDIUM ones of round 2 all fixed.
Round 3 raised four LOW findings: the include pipeline should hand the root variable `$` (F1), `IncludeNames` must strip the leading comment (F2), card 9's N = 1 comparison duplicates the outcome-mapping rows (F3, the continuation of the N = 1 key), and card 1 does not assert the gate's unspent attempt (F4).
None gates, so the loop converges and the four carry as open LOW items.
