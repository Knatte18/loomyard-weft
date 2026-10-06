---
round: 4
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
    rounds: [3, 4]
    status: open
    class: decision
    severity: MEDIUM
  - key: card4-initstate-callers-unlisted
    rounds: [4]
    status: open
    class: scope
    severity: BLOCKING
  - key: card7-stencil-reword-leaves-ladder-false
    rounds: [4]
    status: open
    class: consistency
    severity: MEDIUM
  - key: card7-recover-batch-refusal-envelope
    rounds: [4]
    status: open
    class: decision
    severity: MEDIUM
  - key: card8-doc-go-warning-stale
    rounds: [4]
    status: open
    class: scope
    severity: MEDIUM
  - key: card11-skipgit-remote-update
    rounds: [4]
    status: open
    class: decision
    severity: MEDIUM
  - key: card4-w2c-dropped-unlicensed
    rounds: [4]
    status: open
    class: scope
    severity: MEDIUM
  - key: card11-tenth-executor-tallies-stale
    rounds: [4]
    status: open
    class: consistency
    severity: LOW
  - key: card10-head-vs-hex-contract
    rounds: [4]
    status: open
    class: consistency
    severity: NIT
---

Every round 1 and round 2 key stays resolved.
Round 3's four other open keys do not reappear in round 4's fresh review, so they are resolved.
The round 3 key on the Publish rationale reappears as F7, now a decision-class finding about the `cli-drive` Shared Decision not matching what D8 requires.

Round 4 raises nine findings, none design-class.
The BLOCKING one is F1: card 4 changes `initState` but omits two other webstercli test files that call it, so the plan's own vet fails.
The MEDIUM ones are card 7's stencil reword (F2) and refusal envelope (F3), card 8's stale `doc.go` (F4), card 11's `SkipGit` silence (F5), card 4's unlicensed W2c removal (F6) and the `cli-drive` rationale (F7).
F8 is a LOW stale-tally sweep for card 11, and F9 is a NIT about card 10 passing `HEAD` to a hex-only function.
