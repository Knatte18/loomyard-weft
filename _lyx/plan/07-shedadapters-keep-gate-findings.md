# Card 7 — shedadapters-keep-gate-findings

**Create:**
- `plan:internal/shedadapters#MultiLLMProducer.KeepGateFindings` -> `func (p *MultiLLMProducer) KeepGateFindings(path string) *MultiLLMProducer`
- `plan:internal/shedadapters#clearGateFindings` -> `func clearGateFindings(path string)`

**Edit:**
- `internal/shedadapters#gateFailedReason` -> `func gateFailedReason(gate *shuttleengine.GateOutcome, findingsPath string) string`
- `internal/shedadapters#MultiLLMProducer`
- `internal/shedadapters#MultiLLMProducer.mapOutcome`
- `internal/shedadapters#BurlerHalfDeps`
- `internal/shedadapters#BurlerFixProducer.fixedExit`
- `internal/shedadapters#BurlerReviewProducer.acceptedExit`
- `internal/shedadapters#TestGateFailedReason_NamesAttemptsAndFindingsPath`
- `internal/shedadapters#TestMultiLLMProducer_MapsTheLeadResult`
- `internal/shedadapters#TestBurlerFixProducer_Call_GateFailedAndChangedReviewArchiveTheWholeRound`
- `internal/shedadapters#TestBurlerReviewProducer_Call_AcceptedReviewIsRecorded`
- `internal/shedadapters/doc.go`

**Intent:** D4's producer side: a gated producer told its row's findings-file path writes the exhausted gate's findings there, names that file in its Stuck reason, and removes it on a call whose gate outcome passed.

`gateFailedReason` gains the told path.
A terminal outcome keeps returning `gate.Reason` and writes nothing, since its explanation already rides the reason.
Otherwise, with a non-empty `findingsPath`, it creates the path's directory, writes `gate.Findings` there, overwriting an earlier file, and returns `gate did not pass after <n> attempts; findings: <findingsPath>`.
A failed write is logged at Warn and the reason says `findings could not be kept at <findingsPath>: <cause>; findings: <text>` in place of the path, the text being `gate.Findings` cut to one line of at most 400 characters, so the halt still carries what the gate found.
An empty `findingsPath`, a producer built with none told, keeps today's text naming `gate.FindingsPath`.

`clearGateFindings` removes the file at a non-empty path; an absent file is no error, and another removal failure is logged at Warn, changing no verdict.

`MultiLLMProducer` gains a `gateFindingsPath` field, set by `KeepGateFindings`, which returns the producer so an entry chains it onto either constructor; `BurlerHalfDeps` gains `GateFindingsPath string`, documented the same way.
`MultiLLMProducer.mapOutcome` passes `p.gateFindingsPath` to `gateFailedReason` on a failed chair gate, and calls `clearGateFindings` on the Done path when the chair's gate is non-nil and passed.
`BurlerFixProducer.fixedExit` passes its deps' `GateFindingsPath` to `gateFailedReason` on a failed gate and clears the file on the path past it when the gate is non-nil, which has passed there.
`BurlerReviewProducer.acceptedExit` clears its deps' file when the review's gate is non-nil and passed, so the review row, which never returns Done, clears it too.
`WebsterProducer` carries its engine's own Stuck reason and no gate findings path, so it is unchanged.

The header of `gateattempts.go` names the findings-file helpers beside the attempt and span helpers.

Tests:
- `TestGateFailedReason_NamesAttemptsAndFindingsPath` becomes table-driven: with a temp-dir path the reason names that path and the file holds `gate.Findings`; a second exhausted outcome over the same path overwrites the file with its own text; with an empty path the reason names `gate.FindingsPath` as today; a terminal outcome returns its reason and writes no file; a path under a regular file fails the write and the reason says the findings could not be kept there and carries the findings text.
  Its table also drives `clearGateFindings`: an existing file is removed, an absent file is no error, and a path that is a non-empty directory fails the removal, which is logged and leaves the directory.
- `TestMultiLLMProducer_MapsTheLeadResult` gains a non-terminal failed-gate row, with no `Reason` and `Findings` set, told a findings path through `KeepGateFindings`: it asserts the file holds the text and the reason names it; the existing terminal row keeps its reason and gains the assertion that no file is written.
  A passed-gate row with a pre-written file at the path asserts the file is gone.
- `TestBurlerFixProducer_Call_GateFailedAndChangedReviewArchiveTheWholeRound` gains a non-terminal failed-gate row, with no `Reason` and `Findings` set, told `GateFindingsPath`, which asserts the file and the reason, and a new row whose fix carries a passed gate over a pre-written file, which asserts the file is removed; the existing terminal row keeps `verify is red`.
- `TestBurlerReviewProducer_Call_AcceptedReviewIsRecorded`, whose accepted review carries a passed gate, is told `GateFindingsPath` over a pre-written file and asserts the file is gone.

`doc.go` says every gated producer told a findings path keeps an exhausted gate's findings in that file, names it in the Stuck reason, and removes it on a call whose gate passed.

**ImpactSummary:** `gateFailedReason`'s two production callers are on this card; the new field and method are additive, and a producer told no path behaves as today.
