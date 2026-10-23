# Card 19 — landing-drops-failure-routes

**Delete:**
- `internal/landingshed#FailureRoute`
- `internal/loomcli/darnclassify.go`
- `internal/loomcli/darnclassify_test.go`

**Edit:**
- `internal/landingshed#Deps`
- `internal/landingshed#verifyGate`
- `internal/landingshed#newVerifyGate`
- `internal/landingshed#verifyGate.recordFailure` -> `func (g verifyGate) recordFailure(kind verifytree.FailureKind, row string, result verifytree.Result) bool`
- `internal/landingshed#verifyGate.check`
- `internal/landingshed#verifyGate.checkPublishVerify`
- `internal/landingshed#verifyGate.verdict` -> `func (g verifyGate) verdict(result verifytree.Result, what, producer, parentBranch string) (string, error)`
- `internal/landingshed#TestVerifyGate_CheckMapsVerifyResult`
- `internal/landingshed#TestMergeIn_PlanVerifyFailureRecord`
- `internal/landingshed#TestPublishVerify_FailureRecord`
- `internal/loomcli#verifySource`
- `internal/loomcli#planVerifySource`
- `internal/loomcli#darnVerifySource`
- `internal/loomcli#landingDeps`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`
- `internal/loomcli#TestLandingDeps_VerifyCommandReadsPlanAtCallTime`
- `internal/testkit/envkit#nilLegal`
- `internal/testkit/envkit#LandingDeps`

**Extends:**
- `internal/landingshed/doc.go`

**Intent:** Remove the failure record's routing (D11): `ClassifyVerifyFailure`, `VerifyFailureTarget` and the target's way-forward text go, and no failed gate names a row to go back to.
`Deps` drops `ClassifyVerifyFailure`, `VerifyFailureTarget` and `VerifyFailedWayForward`; `verifyGate` drops `failedWayForward`, `failureTarget` and `classify`, and `FailureRoute` goes.
`recordFailure` writes the record with no target and returns only whether it wrote it, and `verdict` loses its route argument: a failed `landing` or `publish` call's reason ends with the fix-forward clause, "fix forward on the task branch, then resume", the clause the fixer replaces at Merge-In and Finalize.
loomcli's `verifySource` drops `failedWayForward`, `failureTarget` and `classify`; `planVerifySource` and `darnVerifySource` stop setting them, `landingDeps` stops copying them, and darn's classifier file and its test go.
The tests asserting a told route, a target or a classifier lead drop those rows and assert the fix-forward clause instead; the drift test stops skipping `ClassifyVerifyFailure`; `envkit` drops the seam.

`doc.go` drops the target, way-forward and classifier paragraphs of the failure record.

**ImpactSummary:** Three Deps seams and the darn classifier go; they were filled only by loomcli's verify sources and the test kit, and no reader of the record's target remains after cards 14 to 18.
