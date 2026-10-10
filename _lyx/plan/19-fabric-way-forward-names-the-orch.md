# Card 19 — fabric-way-forward-names-the-orch

**Edit:**
- `internal/websterengine#refuseMidMerge`
- `internal/websterengine/gitwrap_test.go`
- `internal/webstercli#fabricSyncWayForward`
- `internal/webstercli#websterCLI.resetCmd`
- `internal/webstercli/verbs_test.go`
- `internal/webstercli/cli_test.go`
- `internal/webstercli/acceptaudit_test.go`
- `internal/loomcli#pushFailureWayForward`
- `internal/loomcli#decisionVerb`
- `internal/loomcli#loomCLI.refuseOverUnfinishedMerge`
- `internal/loomcli/commitrecords_test.go`
- `internal/loomcli/decision_test.go`
- `internal/loomcli/start_midmerge_test.go`
- `internal/landingshed#LoadConfig`
- `internal/landingshed#preMerge.clear`
- `internal/landingshed/config_test.go`
- `internal/landingshed/premerge_test.go`
- `internal/preflightshed#wayForward`
- `internal/preflightshed/wayforward_test.go`
- `internal/reedengine#Engine.validateNaming`
- `internal/reedengine/strand_test.go`
- `internal/boardengine#LoadConfig`
- `internal/boardengine/config_test.go`
- `internal/gateslot#ErrConfigAbsent`
- `internal/gateslot/config_test.go`
- `internal/gatecli#runTest`
- `internal/gatecli/test_integration_test.go`
- `internal/hubgeom/webstergeom_test.go`
- `//contracts/specs/refusal-spec.md`

**Intent:** Add to every refusal-spec row and every error text whose way forward names a fabric verb the guard refuses that the orch runs the verb when the refusing session cannot (D7).
The set is derived by searching the spec and the Go error texts of the modules a refused session runs (webster, loom, shed, gate, reed, board, landing, preflight) for `lyx fabric`: the merge-in-progress texts (`refuseMidMerge`, `preMerge.clear`, `refuseOverUnfinishedMerge`), the fabric-sync and parent-branch texts (`fabricSyncWayForward`, `resetCmd`), the commit and push texts (`decisionVerb`, `pushFailureWayForward`), the reconcile texts (`LoadConfig` in landingshed and boardengine, `ErrConfigAbsent`, `runTest`), the preflight steps (`wayForward`), and the shortname and slug texts (`Engine.validateNaming`).
Each text gains the one clause, worded once: `; a session lyx refuses the verb from reports status: FAILED and the orch runs it`.
The spec's rows that name those verbs gain the same clause, and its `What a way forward is` section states it as the rule for a fabric verb the role guard refuses.
`fabricengine`'s own texts stay as they are: a refused session reaches them only through a verb the guard refuses first.
Every test asserting one of the reworded texts is listed above and asserts the new wording; `TestRefuseMidMerge` in `gitwrap_test.go`, the `wantWayForward` calls in webstercli's tests, `TestLoadConfig_NotInitialized`, the preflight way-forward table and the gate's absent-config row are the exact-string ones, and the rest match substrings the clause leaves intact.

**ImpactSummary:** Thirteen error texts across eight modules gain one trailing clause; no signature changes and the hubgeom test asserts a text fabricengine still owns.
