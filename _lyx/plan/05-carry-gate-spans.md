# Card 5 — carry-gate-spans

**Create:**
- `plan:internal/shedadapters#gateSpansOf` -> `func gateSpansOf(gate *shuttleengine.GateOutcome) []shedengine.GateSpan`

**Edit:**
- `internal/shedadapters#MultiLLMProducer.mapOutcome`
- `internal/shedadapters#BurlerFixProducer.fixedExit`
- `internal/shedadapters#BurlerReviewProducer.acceptedExit`
- `internal/shedadapters#WebsterProducer.Call`
- `internal/shedadapters#TestMultiLLMProducer_MapsTheLeadResult`
- `internal/shedadapters#TestBurlerFixProducer_Call_FinishedFixIsRecorded`
- `internal/shedadapters#TestBurlerFixProducer_Call_GateFailedAndChangedReviewArchiveTheWholeRound`
- `internal/shedadapters#TestBurlerReviewProducer_Call_AcceptedReviewIsRecorded`
- `internal/shedadapters#TestBurlerReviewProducer_Call_ProbeAndResume`
- `internal/shedadapters#TestWebsterProducer_OutcomeDone`
- `internal/shedadapters#TestWebsterProducer_OutcomeStuck`
- `internal/shedadapters/doc.go`

**Uses:**
- `internal/burlerengine#ReviewResult`
- `internal/websterengine#RunResult`

**Intent:**
Every row whose producer runs a gated shuttle run copies that run's gate count and spans into its history entry (D5).
`gateSpansOf`, in `gateattempts.go` beside `gateAttemptsPointer`, flattens a gate outcome's entries into `shedengine.GateSpan` values: one per span, in entry order and, within an entry, span order, each carrying the entry's `Name` as `Gate`, its kind's string (`GateSpanGate` maps to `GateSpanKindGate`, and so on) and its start and end as RFC 3339 UTC.
A nil outcome gives nil, and an outcome whose entries carry no spans gives nil too, so an ungated call and a call whose shuttle recorded nothing keep the field off disk.
The file's header comment names both helpers and all four carriers.

The carriers set `OutputPointer.GateSpans` wherever they set `GateAttempts` from the same outcome:
- `MultiLLMProducer.mapOutcome`, which every seat-table writer row runs through (Discussion-Write, Plan-Write, PR-Rework, Describe, Darn-Write), on its done and its gate-failed Stuck pointers; every writer's decorator passes the session pointer on but PR-Rework's, which card 6 fixes.
- `BurlerFixProducer.fixedExit`, on its gate-failed Stuck and its finished-fix hand-on pointers.
- `BurlerReviewProducer.acceptedExit`, a new carrier: its hand-on pointer carries `gateAttemptsPointer(result.Gate)` and `gateSpansOf(result.Gate)` from the reviewer's review-parse gate, for a spawned and a resumed review alike.
- `WebsterProducer.Call`, a new carrier: its done and its stuck pointers carry the count and spans of `RunResult.Gate`, Merriam's gate, verify entry included.

Tests extend the existing table rows and assertions of each producer:
- `TestMultiLLMProducer_MapsTheLeadResult`: the passing-gate row's result carries two entries, one with a gate and a held span and one with a gate span, and the failing-gate row's entry carries a gate and a rework span, so each kind's mapping is reached; the wanted pointers carry the converted spans in entry order, each with its entry's name and its kind's string.
  A new row, "done with a gate whose entries recorded no spans", wants the attempt count and a nil span list; the ungated "done" row keeps a nil list.
- `TestBurlerFixProducer_Call_FinishedFixIsRecorded` and `TestBurlerFixProducer_Call_GateFailedAndChangedReviewArchiveTheWholeRound`: the scripted fix result's gate carries a span, and the pointer carries it.
- `TestBurlerReviewProducer_Call_AcceptedReviewIsRecorded`: the scripted review result carries a review gate with one attempt and a rework span, and the pointer carries attempt count 1 and the span, replacing today's assertion of no gate attempts.
- `TestBurlerReviewProducer_Call_ProbeAndResume` gains a gate column: the "a live reviewer is resumed and accepted" row's resumed result carries a review gate and its pointer carries the gate's count and spans, and the "not live proceeds to the spawn" row's spawned result carries none and its pointer carries neither.
- `TestWebsterProducer_OutcomeDone` and `TestWebsterProducer_OutcomeStuck`: the run seam's result carries a gate, and the pointer carries its count and spans.

`doc.go`'s producer passages name the review row and the Webster row as gate carriers.

**ImpactSummary:** four producers in `internal/shedadapters` gain one more pointer field from an outcome they already read; two of them start carrying `GateAttempts` too, which changes no routing since shed only records it.

**Commit:** 5: shedadapters: every gated row carries its gate spans
