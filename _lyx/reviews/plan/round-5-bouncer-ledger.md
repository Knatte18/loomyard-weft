---
round: 5
ledger:
  - key: card1-rubric-stencil-unlicensed
    rounds: [1]
    status: resolved
  - key: card1-check-tally-sweep-incomplete
    rounds: [1]
    status: resolved
  - key: card1-fixture-reorder-unnamed
    rounds: [1]
    status: resolved
  - key: cobra-validate-drives-repeated
    rounds: [1]
    status: resolved
  - key: card3-spec-mention-open
    rounds: [1]
    status: resolved
  - key: card4-unknown-role-test-not-deleted
    rounds: [1]
    status: resolved
  - key: card6-test-duplicates-existing
    rounds: [1]
    status: resolved
  - key: card8-stuck-strand-claim-wrong
    rounds: [1]
    status: resolved
  - key: card8-warning-outcome-wording
    rounds: [1]
    status: resolved
  - key: card9-commitsnotin-backend
    rounds: [1]
    status: resolved
  - key: remoteonlycommits-needs-own-card
    rounds: [1]
    status: resolved
  - key: card10-checkout-failed-refusal-untested
    rounds: [1]
    status: resolved
  - key: card10-failed-checkout-envelope
    rounds: [1]
    status: resolved
  - key: card10-stale-docs
    rounds: [1]
    status: resolved
  - key: verify-skips-loomcli-tmux
    rounds: [1]
    status: resolved
  - key: leased-update-no-lease-lost-sentinel
    rounds: [2]
    status: resolved
  - key: gitrepo-boundary-test-method-set
    rounds: [2]
    status: resolved
  - key: remote-force-update-no-gated-executor
    rounds: [2]
    status: resolved
  - key: card1-runlevel-backward-uses-tests
    rounds: [2]
    status: resolved
  - key: card8-paused-outcome-omitted
    rounds: [2]
    status: resolved
  - key: card6-done-row-repeats-tests
    rounds: [2]
    status: resolved
  - key: card11-reset-rows-repeat-tests
    rounds: [2]
    status: resolved
  - key: card3-whole-plan-row-repeats-scope-test
    rounds: [2]
    status: resolved
  - key: cards2-3-check-ids-no-spec-row
    rounds: [2]
    status: resolved
  - key: card11-lease-rejected-test-unreachable
    rounds: [3]
    status: resolved
  - key: card5-recorded-batch-test-misdescribed
    rounds: [3]
    status: resolved
  - key: card4-stencil-line191-model-injection
    rounds: [3]
    status: resolved
  - key: card12-landing-closure-untested
    rounds: [3]
    status: resolved
  - key: cli-drive-publish-rationale-wrong
    rounds: [3, 4, 5]
    status: open
    class: consistency
    severity: NIT
  - key: card4-initstate-callers-unlisted
    rounds: [4]
    status: resolved
  - key: card7-stencil-reword-leaves-ladder-false
    rounds: [4]
    status: resolved
  - key: card7-recover-batch-refusal-envelope
    rounds: [4]
    status: resolved
  - key: card8-doc-go-warning-stale
    rounds: [4]
    status: resolved
  - key: card11-skipgit-remote-update
    rounds: [4, 5]
    status: open
    class: consistency
    severity: LOW
  - key: card4-w2c-dropped-unlicensed
    rounds: [4]
    status: resolved
  - key: card11-tenth-executor-tallies-stale
    rounds: [4]
    status: resolved
  - key: card10-head-vs-hex-contract
    rounds: [4]
    status: resolved
  - key: card4-sandbox-suite-comment-stale
    rounds: [5]
    status: open
    class: scope
    severity: LOW
---

Rounds 1 to 3 are fully resolved.
Round 4's BLOCKING and MEDIUM findings (`initState` callers, card 7 stencil and refusal envelope, card 8 `doc.go`, W2c) do not reappear in round 5's fresh review and are resolved.
Round 5 raises three non-gating findings.
The `cli-drive` Publish key has narrowed from a MEDIUM decision finding to a NIT about card 12 sitting in `Applies to`.
The `SkipGit` key narrowed from MEDIUM decision to LOW consistency, since D7 licenses the choice and only the rationale and a `doc.go` line are missing.
`card4-sandbox-suite-comment-stale` is new: `tools/sandbox/suite.go` still names model escalation after card 4 removes W2.
