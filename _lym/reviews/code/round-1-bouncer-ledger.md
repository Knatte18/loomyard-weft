---
round: 1
ledger:
    - key: tmuxrec-test-file-names-two-tiers
      rounds:
        - 1
      status: resolved
      class: design
      severity: BLOCKING
      fix: mechanical
    - key: tmuxrec-kit-asserts-beyond-setup
      rounds:
        - 1
      status: resolved
      class: design
      severity: BLOCKING
      fix: design-wrong
    - key: replay-depends-on-host-path-and-os
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: design-wrong
    - key: replay-stale-read-after-mutation
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: design-wrong
    - key: replay-ignores-call-stdin
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: design-wrong
    - key: registration-check-loosened
      rounds:
        - 1
      status: open
      class: consistency
      severity: MEDIUM
      fix: mechanical
    - key: replay-refusals-untested
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: placeholding-corrupts-recordings
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: design-wrong
    - key: placeholding-lacks-tier0-coverage
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: pane-end-poll-ends-silently
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: replay-unused-recorded-changes-unnoticed
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: design-wrong
    - key: unlicensed-pattern-paragraph
      rounds:
        - 1
      status: open
      class: scope
      severity: LOW
      fix: mechanical
    - key: overview-kit-list-lacks-tmuxrec
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
    - key: summary-names-tooling-and-understates-blast-radius
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
    - key: empty-scrub-panic-names-wrong-constructor
      rounds:
        - 1
      status: open
      class: consistency
      severity: NIT
      fix: mechanical
---
First round, so every finding is new and open.
Most sit in the new `internal/testkit/tmuxrec` kit:
its tier and assertion placement, the faithfulness of its replay and placeholding, and the coverage of both.
The rest concern a loosened registration check in `landingshed`, documents, the change description and a panic message.
