---
round: 3
ledger:
  - key: push-drop-has-no-recorder
    rounds: [1, 2, 3]
    status: open
    class: scope
    severity: BLOCKING
  - key: pull-first-drop-recordless-caller-unlogged
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
  - key: bolt-push-drop-foreign-board-test-needs-export-shim
    rounds: [2]
    status: resolved
  - key: foreign-board-fixture-construction-unstated
    rounds: [2]
    status: resolved
  - key: scenario-label-assertion-vs-fixed-identity
    rounds: [2]
    status: resolved
  - key: replay-already-applied-commit-conflicts
    rounds: [2]
    status: resolved
  - key: prerun-pull-first-drops-stencils-seed
    rounds: [1]
    status: resolved
  - key: bolt-tests-not-on-hubforge
    rounds: [1]
    status: resolved
  - key: isnoserver-misses-absent-socket
    rounds: [1]
    status: resolved
  - key: drop-gate-binds-only-base-name
    rounds: [1]
    status: resolved
  - key: tracked-dirtiness-refuses-normal-state
    rounds: [1]
    status: resolved
  - key: card7-repeats-issseedcommit-refuse-rows
    rounds: [1]
    status: resolved
  - key: card7-bundles-three-units
    rounds: [1]
    status: resolved
  - key: kindcommitsdropped-missing-from-oracle
    rounds: [1]
    status: resolved
  - key: bolt-type-edit-target-no-change
    rounds: [1]
    status: resolved
  - key: card3-impact-claims-writer-keys-early
    rounds: [1]
    status: resolved
---

Round 1 raised eleven findings, round 2 five, round 3 two.

The one carried key is `push-drop-has-no-recorder`: a push caller whose mutation envelope omits the commits a seed drop discarded.
Round 2 found it for `lyx fabric reconcile`; round 3 finds the last caller, `shortname`, that card 13 lists as recordless although `runShortname` holds a live `Mutations`.
Its class moved from `design` to `scope` as the defect narrowed to one caller's card edit list, and it stays BLOCKING.

Round 3 added `pull-first-drop-recordless-caller-unlogged`, a LOW consistency gap: cards 8 and 10 do not state the record `PullThenCommitWritten` receives, so a drop on their path is neither enveloped nor logged.

Every other round 1 and round 2 key was not re-raised and is resolved.
