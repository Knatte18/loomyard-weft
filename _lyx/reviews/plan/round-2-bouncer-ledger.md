---
round: 2
ledger:
  - key: push-drop-has-no-recorder
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
  - key: bolt-push-drop-foreign-board-test-needs-export-shim
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
  - key: foreign-board-fixture-construction-unstated
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: scenario-label-assertion-vs-fixed-identity
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: replay-already-applied-commit-conflicts
    rounds: [2]
    status: open
    class: design
    severity: LOW
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

Round 1 raised eleven findings; round 2 raised five.
Round 2 did not re-raise ten of the round 1 keys, and its focus-directive verdicts cleared the pre-run pull-first, the drop gate's bound, the dirty-tracked refusal and `IsNoServer`, so those ten are resolved.

The one carried key is `push-drop-has-no-recorder`.
Round 1 had it as a MEDIUM `decision` finding; round 2 found the sharper form: `Bolt.Push` appends to a throwaway `Mutations`, so `lyx fabric reconcile`, whose handler card 11 edits and whose two pushes go through `Bolt.Push`, resets `_board` without the drop showing in its envelope.
It is now BLOCKING `design`.

Four new keys came in round 2, none gating: the missing `export_test.go` shim for card 7's foreign-board test, that fixture's unstated construction, the scenario's label assertion against the fixed identity, and the empty-pick case in the cherry-pick replay.
