# Card 26 — Merge-In and Publish record the failure trace

**Edit:**
- `internal/landingshed#Deps`
- `internal/landingshed#failureRecorder`
- `internal/landingshed#verifyGate.recordFailure`
- `internal/landingshed#NewPublish`
- `internal/landingshed#NewMergeIn`
- `internal/landingshed#TestMergeIn_PlanVerifyFailureRecord`
- `internal/landingshed#TestPublishVerify_FailureRecord`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`
- `internal/loomcli#landingDeps`
- `internal/testkit/envkit#nilLegal`
- `internal/landingshed/doc.go`

**Uses:**
- `plan:internal/failuretrace#NewTracer`
- `plan:internal/verifytree#FailureTrace`

**Intent:**
D9: when Merge-In or Publish writes the Publish failure record on a loom run, Go also traces the failure; `landingshed` gets the trace as a told seam (PATTERN-told-geometry), wired in `internal/loomcli`.

`landingshed.Deps` gains `TraceFailure func(log, head string) (verifytree.FailureTrace, error)`; nil traces nothing.
`failureRecorder` gains the seam, and `NewPublish` and `NewMergeIn` set it from `deps.TraceFailure` where they build the recorder.
`recordFailure`, after it has read the log and HEAD and before it writes the record, calls the seam with the log and the record's `Head` when the seam is set and both are non-empty, and copies the trace's cards, later range and untraced locations onto the record.
A trace error, such as no plan, no Webster state or a quarry error, is logged at Warn and leaves the record without the fields; it never changes the Stuck verdict, the route or the record's other fields.

`loomcli.landingDeps` sets `TraceFailure` to `failuretrace.NewTracer(l.WorktreePath(), l.AnchorPath())` for the loom recipe and leaves it nil for darn, which has no plan.
`envkit`'s `nilLegal` lists `Landing.TraceFailure`, since nil is the darn default.
`internal/landingshed/doc.go`'s failure-record paragraph names the trace fields, the seam and the failure rule.

`TestMergeIn_PlanVerifyFailureRecord` gains rows: a told trace's cards, later range and untraced land in the written record; a trace error writes the record without them, with the same Stuck reason and target; a nil seam writes today's record; an empty HEAD calls no trace.
`TestPublishVerify_FailureRecord` gains a row: a told trace's fields land in Publish's record, so `NewPublish`'s wiring of the seam is pinned too.
`TestLandingDeps_EveryFieldPopulated` builds loom deps, so its reflection over every field already requires the new seam set; it gains a darn subtest asserting `TraceFailure` is nil.

**ImpactSummary:** one new optional seam on `landingshed.Deps`, set only by `landingDeps` for loom; `recordFailure` is shared by Merge-In and Publish, and with a nil seam or a failing trace it writes exactly today's record.
