---
dispositions: []
---

## Changes

The live gate went red because its `lym gate test` call carried tier1's green record over an unchanged tree and started no test binary.
`lym gate test` gained a `--no-carry` flag that treats every tier of the call as a `carry: false` tier.
The hardener's live call passes it, so a live run always executes its tests.

- `internal/gateslot/tiers.go`: `NoCarryFlag`.
- `internal/gatecli/cli.go`, `tiers.go`, `detachfront.go`, `doc.go`: the flag, its request field, its forwarding to a detached runner and its documentation.
- `internal/loomcli/hardenwiring.go`: `hardenLiveArgs` passes `--no-carry`.
- Tests: a `TestPlanTierRuns` row in `internal/gatecli/tiers_test.go`, and the `hardenLiveArgs` rows in `internal/loomcli/hardenwiring_test.go`.

A follow-up commit split the two comment lines the gate flagged for length, in `internal/gatecli/doc.go` and `internal/gateslot/tiers.go`, into shorter sentences.

Verification: `lym gate test --upto 1` over `gatecli`, `gateslot` and `loomcli` passed in tier0 and tier1.
Sabotage: with the `request.noCarry` line in `planTierRuns` removed, the new row fails; restored.

## Disputed

None.

## Deferred

None.

## Rulings

None.
