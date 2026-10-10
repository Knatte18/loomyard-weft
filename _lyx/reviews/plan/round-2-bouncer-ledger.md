---
round: 2
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
    status: open
    class: scope
    severity: BLOCKING
  - key: card11-shedbuild-every-engine-build-test-missing
    rounds: [2]
    status: open
    class: scope
    severity: BLOCKING
  - key: card2-3-template-include-passes-no-data
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: card6-9-n1-equivalence-pinned-only-via-fake
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1's eleven findings (none `design`) are not re-raised in round 2 and are resolved.
Round 2 raised four new findings.
Two are BLOCKING scope gaps where a card's own gate fails on an existing test it does not list: card 10 on the Env seam-completeness tests in `envkit` and `battencli`, card 11 on shedbuild's every-registered-engine build test.
The `design` MEDIUM is the bare `{{template "name"}}` include in cards 2 and 3, which executes with nil data.
The LOW is that N = 1 is pinned only against the fake seat runner.
