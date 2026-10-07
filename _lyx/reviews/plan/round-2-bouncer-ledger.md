---
round: 2
ledger:
  - key: card13-clock-rename-unlisted-tests
    rounds: [1]
    status: resolved
  - key: card10-triagepath-dead-field
    rounds: [1]
    status: resolved
  - key: card10-gitattributes-dead-line
    rounds: [1]
    status: resolved
  - key: card11-stale-asking-prose
    rounds: [1]
    status: resolved
  - key: card5-three-units
    rounds: [1]
    status: resolved
  - key: card5-expired-shell-offset-unspecified
    rounds: [1]
    status: resolved
  - key: card5-notified-offset-write-failure
    rounds: [1]
    status: resolved
  - key: card12-driver-stencil-tail-contradiction
    rounds: [1]
    status: resolved
  - key: card5-hold-test-duplicates
    rounds: [1]
    status: resolved
  - key: card14-scenario-repeats-card11-smoke
    rounds: [1]
    status: resolved
  - key: card13-corpus-cases-no-reading-source
    rounds: [1]
    status: resolved
  - key: card1-fork-label-untested
    rounds: [1]
    status: resolved
  - key: card1-transcript-line-pointers
    rounds: [1]
    status: resolved
  - key: card5-asking-tests-enumeration-misses-four
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: card13-driver-stencil-nothing-more-verbatim
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: card14-public-clock-seam-bundled
    rounds: [2]
    status: open
    class: design
    severity: LOW
  - key: card5-impactsummary-package-tally
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised thirteen findings, one BLOCKING and five MEDIUM.
Round 2's fresh review did not re-raise any of them, so all are carried as resolved.
Round 2 raised four new findings, none recurring.
The gating one is F1, the test enumeration in card 5, which misses four tests including one in `attach_test.go` outside the card's targets.
F2 is the driver stencil's "nothing more" and "verbatim" sentences contradicting the added tail.
F3 asks to split the public clock seam out of the corpus card, and F4 is a stale package tally in card 5's `ImpactSummary`.
