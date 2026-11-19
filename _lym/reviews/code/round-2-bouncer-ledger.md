---
round: 2
ledger:
  - key: code-ask-misses-rounds-own-review
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: parent-agreement-not-bound-to-rulings
    rounds: [1]
    status: resolved
  - key: parent-gate-keys-open-to-local-layer
    rounds: [1]
    status: resolved
  - key: drift-key-name-differs-from-entry
    rounds: [1]
    status: resolved
  - key: judge-parent-wait-outlives-judge
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: production-seat-construction-untested
    rounds: [1]
    status: resolved
  - key: judge-table-agreement-entry-untested
    rounds: [1]
    status: resolved
  - key: contained-tests-lack-keep-marker
    rounds: [1]
    status: resolved
  - key: gate-rows-missing-repost-resolve-race
    rounds: [1]
    status: resolved
  - key: each-round-seat-round-one-unpinned
    rounds: [1]
    status: resolved
  - key: judge-seat-wiring-untested
    rounds: [1]
    status: resolved
  - key: stale-answer-refusal-names-decision-record
    rounds: [1]
    status: resolved
  - key: parentask-doc-stale-for-drift-judge
    rounds: [1]
    status: resolved
  - key: foreign-ask-latest-rerolls-discussion-ask
    rounds: [1]
    status: resolved
  - key: judge-parent-block-embedded-in-go
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
---

Round 1 raised fourteen findings; round 2 resolved twelve of them and carried two forward.
F15 carries the round-1 ask-timing key: the parent-findings refusal still runs only on the Bouncer's fresh harvest, not on its recovery paths.
F17 carries the uncapped judge wait: command mode never sets the judge's start time.
F16 is new this round: the judge's parent-review prompt block is embedded in Go instead of a stencil fragment.
