# Card 2 — landingshed-failure-route

**Create:**
- `internal/landingshed#FailureRoute`

**Edit:**
- `internal/landingshed#Deps`
- `internal/landingshed#verifyGate`
- `internal/landingshed#newVerifyGate`
- `internal/landingshed#verifyGate.recordFailure` -> `func (g verifyGate) recordFailure(ctx context.Context, kind verifytree.FailureKind, row string, result verifytree.Result) (FailureRoute, bool)`
- `internal/landingshed#verifyGate.verdict` -> `func (g verifyGate) verdict(result verifytree.Result, what, producer, parentBranch string, route FailureRoute, recorded bool) (string, error)`
- `internal/landingshed#verifyGate.check`
- `internal/landingshed#verifyGate.checkPublishVerify`
- `internal/landingshed#TestPublishVerify_FailureRecord`
- `internal/landingshed#TestMergeIn_PlanVerifyFailureRecord`
- `internal/landingshed/doc.go`

**Uses:**
- `internal/landingshed#newMergeInFixture`

**Intent:** D1's seam on the landingshed side: Merge-In and Publish choose a failure record's target and the Stuck reason's way forward per failure, through a told classifier, in place of the fixed values.

`FailureRoute` is the classifier's answer, with three fields: `Target`, the row written into the record's target; `WayForward`, the whole clause the reason ends with after `way forward: `; and `Lead`, text the reason puts between `<gate> failed` and ` after merging parent branch`, empty for a failure the task owns.
A not-owned route's lead reads ` outside the task's impacted set, in <packages>,`, so the reason starts `landing gate failed outside the task's impacted set, in <packages>, after merging parent branch "main" (exit code 1)`, and the batten blocked notice, which keeps the first 200 characters of the child's error, carries the not-owned words (card 4).

`Deps` gains `ClassifyVerifyFailure func(ctx context.Context, failure verifytree.PublishFailure) FailureRoute`, documented like the fields around it: called by Merge-In and Publish when they write a failure record, with the record as written except its target, its log path naming the failing run's log; nil keeps `VerifyFailureTarget` and `VerifyFailedWayForward`, loom's value.
`VerifyFailureTarget` and `VerifyFailedWayForward` stay, documented as the values a nil classifier keeps.
`verifyGate` gains a `classify` field beside `failedWayForward` and `failureTarget`; `newVerifyGate` copies `deps.ClassifyVerifyFailure` into it.

`recordFailure` builds the record, its target left empty, then picks the route: `classify(ctx, failure)` when the field is set, else `FailureRoute{Target: g.failureTarget, WayForward: g.failedWayForward}`.
It writes `route.Target` as the record's target and returns the route with whether it wrote the record; a result that writes no record returns the zero route and false, as today.
`check` and `checkPublishVerify` pass their `ctx` and hand the route and the flag to `verdict`.
`verdict`'s failed-with-exit-code branch uses `route.WayForward` where it used `g.failedWayForward`, and inserts `route.Lead` after `failed`; with no record or an empty way forward it keeps the fix-forward clause, so a zero route renders today's text exactly.

`TestPublishVerify_FailureRecord` gains a row whose fixture sets `fx.p.gate.classify` to a fake that records the failure it was handed and returns `FailureRoute{Target: "Merge-In", WayForward: "the told not-owned route", Lead: " outside the task's impacted set, in example.com/m/a,"}`:
the record's target is `Merge-In`, the fake saw the record's kind, row, tests and log path, and the reason holds `publish gate failed outside the task's impacted set, in example.com/m/a, after merging parent branch` and ends with `way forward: the told not-owned route`.
The existing rows keep the nil classifier and their `Told-Row` target, pinning the fixed fallback loom keeps.
`TestMergeIn_PlanVerifyFailureRecord` gains a subtest over `newMergeInFixture` with `fx.m.gate.classify` set to the same kind of fake: a failed landing call at Merge-In writes target `Merge-In` and the reason ends with the fake's way forward.
The subtest sets the field on `fx.m.gate`, never on `fx.gate.gate`, because `newMergeInFixture` copies the `verifyGate` value into the `MergeIn` it builds, so a later write to the fixture's copy never reaches the producer.

`doc.go`'s failure-record paragraph says the record's target and the reason's way forward come from the told `Deps.ClassifyVerifyFailure` when it is set, and from `Deps.VerifyFailureTarget` and `Deps.VerifyFailedWayForward` otherwise, and that a not-owned route's lead names the packages before the merge clause.

The seam only changes which row a record names and which words a reason ends with; every failure still halts the row, so it lets nothing through.

**ImpactSummary:** two private re-signs whose callers are the two gate methods on this card; `Deps` gains one optional field, filled only by card 3's darn wiring.
