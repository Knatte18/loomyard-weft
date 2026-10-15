# Card 4 — surface-gate-outcomes

**Edit:**
- `internal/burlerengine#ReviewResult`
- `internal/burlerengine#reviewResultOf`
- `internal/burlerengine#TestEngine_Review_Outcomes`
- `internal/burlerengine/doc.go`
- `internal/websterengine#RunResult`
- `internal/websterengine#finishMasterDone`
- `internal/websterengine_test#TestRun_DoneOutcome`
- `internal/websterengine/doc.go`

**Intent:**
Two gated runs whose rows copy no gate outcome today surface it, so card 5's adapters can copy their count and spans (D5's carriers).

Burler's review half: `ReviewResult` gains `Gate *shuttleengine.GateOutcome`, a 1:1 passthrough of the reviewer's `shuttleengine.Result.Gate` (the review-parse entry, `ReviewGateEntry`), nil when the reviewer never ran gated, the shape `FixResult.Gate` already has.
`reviewResultOf` copies it from the reviewer's result, so the spawned review (`Engine.Review`) and the resumed one (`Engine.ResumeReview`) both carry it on every result they build from a reviewer result, a non-done outcome included.
`TestEngine_Review_Outcomes` gains a `wantGateReason` column with a nil-gate check: the "a failed review gate is an error" row, whose scripted result carries a gate with reason `spent`, asserts the returned result's `Gate` carries that reason, and the "a reviewer that died" row asserts `Gate` is nil.

Webster: `RunResult` gains `Gate *shuttleengine.GateOutcome`, Merriam's run's gate outcome with the verify gate's entry among its entries.
`finishMasterDone` sets it from the shuttle result on every `RunResult` it returns, the verify-gate demotion to stuck included; every other exit returns no `RunResult`, as today.
`TestRun_DoneOutcome` gains a `gate` case field, which the case's Master result carries, and two rows: a passing gate with one attempt, where `RunResult.Gate` is that outcome and the run is done, and a failing gate, where the run is demoted to stuck and `RunResult.Gate` still carries it.

Each package's `doc.go` names the new field where it describes the result: burlerengine's `ReviewResult` passage beside `FixResult`'s `Gate`, and websterengine's run-exit passage.

**ImpactSummary:** one additive field on each of two result structs, set at the one place each result is built from a shuttle result; existing readers ignore it, and the only consumer arrives in card 5.

**Commit:** 4: burler, webster: surface the reviewer's and Merriam's gate outcome
