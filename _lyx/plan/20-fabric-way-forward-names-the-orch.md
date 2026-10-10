# Card 20 — fabric-way-forward-names-the-orch

**Edit:**
- `internal/websterengine#refuseMidMerge`
- `internal/websterengine/gitwrap_test.go`
- `internal/webstercli#fabricSyncWayForward`
- `internal/webstercli#websterCLI.resetCmd`
- `internal/webstercli/verbs_test.go`
- `internal/webstercli/cli_test.go`
- `internal/webstercli/acceptaudit_test.go`
- `internal/webstercli/reset_test.go`
- `internal/loomcli#pushFailureWayForward`
- `internal/loomcli#decisionVerb`
- `internal/loomcli#loomCLI.refuseOverUnfinishedMerge`
- `internal/loomcli/commitrecords_test.go`
- `internal/loomcli/decision_test.go`
- `internal/loomcli/start_midmerge_test.go`
- `internal/loomengine#LoadConfig`
- `internal/loomengine/config_test.go`
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
- `internal/fabricengine#LoadConfig`
- `internal/fabricengine#ReadOriginFor`
- `internal/fabricengine/config_test.go`
- `internal/hubgeom/geometry_integration_test.go`
- `//contracts/specs/refusal-spec.md`

**Intent:** Add to every refusal-spec row and every error text whose way forward names a fabric verb the guard refuses, and that a refused session can reach, that the orch runs the verb when the refusing session cannot (D7).
The set is derived by searching the spec and the Go error texts for `lyx fabric`, then keeping each text a refused session reaches through a module it runs (webster, loom, shed, gate, reed, board, landing, preflight) or through a fabric reader verb: the merge-in-progress texts (`refuseMidMerge`, `preMerge.clear`, `refuseOverUnfinishedMerge`), the fabric-sync and parent-branch texts (`fabricSyncWayForward`, `resetCmd`), the commit and push texts (`decisionVerb`, `pushFailureWayForward`), the reconcile texts (`LoadConfig` in loomengine, landingshed, boardengine and fabricengine, `ErrConfigAbsent`, `runTest`, and `ReadOriginFor`'s missing-lock text), the preflight steps (`wayForward`), and the shortname and slug texts (`Engine.validateNaming`).
`fabricengine.LoadConfig` is reached through the reader verbs `list`, `pairs` and the dry-run `prune` and `cleanup`, and `ReadOriginFor` through `hubgeom.ResolveParent`, which `lyx webster reset` and every geometry builder call, so both join the set although they live in `fabricengine`.
Each text gains the one clause, worded once: `; a session lyx refuses the verb from reports status: FAILED and the orch runs it`.
The spec's rows that name those verbs gain the same clause, and its `What a way forward is` section states it as the rule for a fabric verb the role guard refuses.
Every other `fabricengine` text naming a fabric verb stays as it is: the merge conclude texts, the junction and adoption texts, the clone and add texts and the checkout retry are reached only through `merge`, `reconcile`, `clone`, `add` or `checkout`, verbs the guard refuses before they run.
Every test asserting one of the reworded texts is listed above and asserts the new wording; `TestRefuseMidMerge` in `gitwrap_test.go`, the `wantWayForward` calls in webstercli's tests, `TestLoadConfig_NotInitialized` in loomengine and landingshed, the preflight way-forward table and the gate's absent-config row are the exact-string ones, and every other listed test, which matches a substring of its text, gains the clause in one of its assertions, so dropping the clause from any reworded text fails a row; `fabricengine`'s `TestLoadConfig` gains the clause among its wanted substrings.
No test reaches `resetCmd`'s unknown-parent refusal today, so `TestResetCmd` gains a row whose parent-branch lookup fails, asserting the refusal names `lyx fabric reconcile` with the clause.
`ReadOriginFor`'s missing-lock text is already asserted through `hubgeom.ResolveParent` in `geometry_integration_test.go`, whose row gains the clause among its wanted substrings, so no new test is added.

**ImpactSummary:** One trailing clause on each way-forward text a refused session can reach, across the webster, loom, shed, gate, reed, board and fabric packages; no signature changes, and every test of those texts is on this card.
