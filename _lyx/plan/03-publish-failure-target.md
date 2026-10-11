# Card 3 — publish-failure-target

**Edit:**
- `internal/verifytree#PublishFailure`
- `internal/verifytree#TestPublishFailureRecord`
- `internal/landingshed#Deps`
- `internal/landingshed#verifyGate`
- `internal/landingshed#newVerifyGate`
- `internal/landingshed#verifyGate.recordFailure`
- `internal/landingshed#TestPublishVerify_FailureRecord`
- `internal/landingshed/doc.go`
- `internal/loomcli#verifySource`
- `internal/loomcli#planVerifySource`
- `internal/loomcli#darnVerifySource`
- `internal/loomcli#landingDeps`
- `internal/loomcli#TestLandingDeps_VerifyCommandReadsPlanAtCallTime`
- `internal/loomcli#TestWire_DarnRecipe`

**Intent:**
The Publish failure record gains the two fields batten's automatic retry (D5) needs to name the target and to tell this halt's record from a stale one.

`verifytree.PublishFailure` gains `Target string` (yaml `target`, omitempty), the producer row a fix is routed back to, and `WrittenAt string` (yaml `written_at`, omitempty), the record's write time in RFC 3339 whole-second UTC, the form shed's history `at` uses.
A record written before this change decodes with both empty.

`landingshed.Deps` gains `VerifyFailureTarget string`, the row the recipe routes a failed Publish verify back to, documented beside `VerifyFailedWayForward`; empty writes no target.
`verifyGate` carries it as `failureTarget`, copied in `newVerifyGate`, and `verifyGate.recordFailure` writes it into `Target` and writes `WrittenAt` from the gate's `now` seam as `now().UTC().Format(time.RFC3339)`.
`internal/landingshed/doc.go`'s failure-record paragraph names the two new fields.

In `internal/loomcli`, `verifySource` gains `failureTarget`: `planVerifySource` sets `loomshed.NameWebsterBurler` and `darnVerifySource` sets `loomshed.NameDarnWrite`, the rows each source's `failedWayForward` already names, and `landingDeps` passes it as `VerifyFailureTarget`.
`TestLandingDeps_EveryFieldPopulated`'s reflection guard then covers the new field without an edit, since it builds the deps over `planVerifySource`.

Tests:
`TestPublishFailureRecord` round-trips a record carrying `Target` and `WrittenAt`;
`TestPublishVerify_FailureRecord` asserts a written record carries the told target and the fake clock's whole-second UTC time;
`TestLandingDeps_VerifyCommandReadsPlanAtCallTime` asserts `VerifyFailureTarget` is `Webster-Burler`, and `TestWire_DarnRecipe` that the darn verify source's target is `Darn-Write`.

**ImpactSummary:** two additive record fields and one additive Deps field; Publish's verdicts and every existing record reader are unchanged.
