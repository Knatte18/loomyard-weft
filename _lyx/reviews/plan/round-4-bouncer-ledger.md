---
round: 4
ledger:
  - key: drop-not-serialised-against-push-rebase
    rounds: [4]
    status: open
    class: design
    severity: MEDIUM
  - key: push-drop-has-no-recorder
    rounds: [1, 2, 3]
    status: resolved
  - key: pull-first-drop-recordless-caller-unlogged
    rounds: [3]
    status: resolved
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

Round 1 raised eleven findings, round 2 five, round 3 two, round 4 one.

Round 4 did not re-raise the recorder gap (`push-drop-has-no-recorder`) or the unlogged recordless pull-first caller, so both are resolved, along with every earlier key.

The one new key is `drop-not-serialised-against-push-rebase`, a MEDIUM design finding: the seed-commit drop in `PushRecorded` and `pullFirst` holds `board.lock` but not the push lock `pull --rebase` runs under, so a concurrent pusher's rebase can interleave with the reset and replay.
First raised this round, so it does not block convergence.
