# gate-runs-once: stopped at batch 08 on a plan conflict

Five of the 17 batches are done: 01, 03, 04, 05 and 06.

- **01 (cards 1, 2):** `gate.yaml`'s `gates` became one key per site with tiers and scope, with a migration, and `lyx gate test --gate <key>` was added with the run record's `gate_key`. Head `32173fbd7`. Deviations: files in `internal/gatecli`, `internal/gateslot`, `internal/landingshed`, `internal/loomcli`, `internal/loomshed`, `internal/webstercli` and `internal/websterengine`. Departures: none.
- **03:** every site's call renders as `--gate <key>`. Head `50cba96f6`. Deviations: `internal/loomcli/wiring_darn_test.go`, `internal/loomshed/verifygate_integration_test.go`. Departures: none. The fork skipped one `loomshed` subtest, the misplaced-marker row, for card 8 to re-enable.
- **04:** the fix rows and the darn writer run their own row key. Head `09c97daab`. Deviations: `internal/loomcli/landingdeps.go`, `internal/loomcli/wiring_darn_test.go`, `internal/loomshed/verifygate_integration_test.go`. Departures: none.
- **05:** Merge-In and Finalize run and record their own row key. Head `c6db660c8`. Deviations: `internal/landingshed` tests and `verifygate.go`, `internal/loomcli/landingdeps_test.go`, `internal/shedrecipe/entries_simple.go`, `internal/testkit/envkit/envkit.go`. Departures: none.
- **06 (cards 6, 7):** `tree_scope_verify` retired for the row key over the touched trees' targets, and a green tier carries over an unchanged clean tree. Head `b37825104`. Deviations: `internal/configcli/hubwide.go`, `internal/darnengine` config and test, `internal/gatecli` `green.go`, `green_test.go`, `run.go`, `scopes_integration_test.go` and `tierrun.go`. Departures: none. A warning noted that card 14 references `internal/gatecli#runGuardStep`, which batch 06 removed.

Batch 08 did not begin. `begin-batch 08` refused with `plan_conflict: tree_moved`: card 14 still names `internal/gatecli#runGuardStep`, which batch 06 replaced with `guardStepRun`.
The plan needs card 14 edited to match the tree, then `lyx webster rebaseline --card 14`, then a resumed run.
