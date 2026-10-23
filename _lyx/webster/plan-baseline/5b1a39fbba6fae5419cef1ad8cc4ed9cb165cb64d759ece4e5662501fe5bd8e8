# Card 46 — landing-failure-record

**Create:**
- `internal/verifytree#LandingFailure`
- `internal/verifytree#WriteLandingFailure`
- `internal/verifytree#ReadLandingFailure`
- `internal/verifytree#RemoveLandingFailure`
- `internal/verifytree#TestLandingFailureRecord`

**Delete:**
- `internal/verifytree#PublishFailure`
- `internal/verifytree#WritePublishFailure`
- `internal/verifytree#ReadPublishFailure`
- `internal/verifytree#RemovePublishFailure`
- `internal/verifytree#publishFailureName`
- `internal/verifytree#TestPublishFailureRecord`

**Edit:**
- `internal/verifytree#Paths`
- `internal/verifytree#NewPaths`
- `internal/verifytree#pruneLogs`
- `internal/verifytree#TestVerify_Scenario`
- `internal/landingshed#failureRecorder`
- `internal/landingshed#verifyGate.recordFailure`
- `internal/landingshed#verifyGate.clearFailure`
- `internal/landingshed#fixerRun.run`
- `internal/landingshed#TestMergeIn_PlanVerifyFailureRecord`
- `internal/landingshed#TestFinalizeVerify_Fail`
- `internal/landingshed#TestVerifyGate_CheckMapsVerifyResult`
- `internal/failuretrace/doc.go`

**Extends:**
- `internal/verifytree/doc.go`

**Intent:** The verifytree Publish failure record is renamed the landing failure record, in its type, accessors, file name and docs (D12), and its target and row fields go (D11).
`PublishFailure` becomes `LandingFailure` without `Target` and `Row`; `WritePublishFailure`, `ReadPublishFailure` and `RemovePublishFailure` become the `Landing` spellings; `Paths.PublishFailure` becomes `Paths.LandingFailure`, and the file is `landing-failure.yaml`.
A record an older build left under `publish-failure.yaml` is ignored: nothing reads that name any more, and `Verify`'s log pruning keeps only the log the landing record names.
Merge-In and Finalize write it on a failed `landing` gate with the gate, tier, failing tests, log, HEAD, merge-in commit and, in a loom run, the plan trace (`TraceFailure`); a passing gate at either row removes it, as does a skipped gate over a tree `verifytree` records as passed, and a later failure overwrites it.
The fixer's prompt names it through `FixerInputs.FailureRecordPath`, and no other reader remains.
The record tests, the scenario test's pruning row and the landing tests read the new spelling, and one row pins that a file left under the old name is neither read nor kept from pruning.
`TestVerifyGate_CheckMapsVerifyResult` gains a row where a skipped gate over a tree the record names as passed removes a present landing failure record.
`internal/failuretrace`'s `doc.go` names the landing failure record where it named the Publish one.

`doc.go` names the landing failure record and its writers.

**ImpactSummary:** One record type and its accessors renamed and two fields dropped; after the earlier cards its readers are the landing gate, the fixer and the log pruning, all listed.
