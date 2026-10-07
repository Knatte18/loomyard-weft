---
round: 3
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
    status: resolved
  - key: card13-driver-stencil-nothing-more-verbatim
    rounds: [2]
    status: resolved
  - key: card14-public-clock-seam-bundled
    rounds: [2]
    status: resolved
  - key: card5-impactsummary-package-tally
    rounds: [2]
    status: resolved
  - key: card5-attach-asking-reentry-test-unlisted
    rounds: [3]
    status: open
    class: scope
    severity: MEDIUM
  - key: card13-driver-stencil-tail-unquoted
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
  - key: card14-clock-rename-typed-edit
    rounds: [3]
    status: open
    class: design
    severity: LOW
  - key: card14-coverage-sentence-overstates
    rounds: [3]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised thirteen findings, one BLOCKING and five MEDIUM, and round 2's fresh review re-raised none of them.
Round 2 raised four findings, and round 3's fresh review re-raised none of those either, so all are carried as resolved.
Round 3 raised four new findings, none recurring.
F1 is a MEDIUM scope finding: card 5 leaves `TestAttach_AskingReentryStartsAtRecordedOffset` failing at its own commit.
F2 asks the driver stencil to quote the tail literally, F3 asks to type the `clock` to `Clock` rename as a `Rename` group, and F4 corrects an overstated coverage sentence in card 14.
No open finding is gating, so the loop converges.
