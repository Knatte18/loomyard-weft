# Card 3 — history-gate-spans

**Create:**
- `internal/shedengine#GateSpan`
- `internal/shedengine#GateSpanKindGate`
- `internal/shedengine#GateSpanKindHeld`
- `internal/shedengine#GateSpanKindRework`

**Edit:**
- `internal/shedengine#HistoryEntry`
- `internal/shedengine#OutputPointer`
- `internal/shedengine#Shed.stepLocked`
- `internal/shedengine#Shed.rerouteStuck`
- `internal/shedengine#appendOrFold`
- `internal/shedengine#cloneHistory`
- `internal/shedengine#gatedOutcomeProducer` -> `func gatedOutcomeProducer(outcome Outcome, outputPath string, gateAttempts *int, gateSpans []GateSpan) *funcProducer`
- `internal/shedengine#TestStep_GateAttempts`
- `internal/shedengine#TestAppendOrFold_Folds`
- `internal/shedengine#TestAppendOrFold_InputUnchanged`
- `internal/shedengine#TestStep_RerouteMovesToTargetAndEndsTargetSegmentEpisode`
- `internal/shedengine#scriptedRerouteProducer`
- `internal/shedengine#TestRun_RerouteCapHaltsOnFourthRerouteAndResumeGrantsThreeMore`
- `internal/shedengine#TestStep_RerouteWithoutOnRerouteFailsRun`
- `internal/shedengine#TestStatus_JSONRoundTrip`
- `internal/shedengine/doc.go`

**Intent:**
The shed history entry stores a call's gate spans as an optional list beside `gate_attempts` (D5).
`GateSpan` (in `status.go`) is a plain stdlib struct, so `shedengine` imports nothing new (PATTERN-shed-producer-seam): `Gate string` (`json:"gate"`, the entry's name as recorded), `Kind string` (`json:"kind"`, one of the three `GateSpanKind*` constants), `Start string` and `End string` (`json:"start"`, `json:"end"`, RFC 3339 UTC with fractional seconds, `time.RFC3339Nano`, so a sub-second evaluation keeps its length).
`HistoryEntry` gains `GateSpans []GateSpan` with `json:"gate_spans,omitempty"`, and `OutputPointer` gains `GateSpans []GateSpan`, carried straight through like `GateAttempts`.
The field is additive and tolerant (PATTERN-tolerant-records): an entry written before it reads with a nil list, and a span object carrying an unknown key still decodes.
Writes keep going through `internal/state`.

Every entry kind that carries `GateAttempts` today carries the spans: the verdict entry `Shed.stepLocked` builds, and the three entries `Shed.rerouteStuck` builds (the Stuck of a row with no `OnReroute`, the reroute entry and the reroute-cap halt).
Folding keeps today's test, which compares `GateAttempts` and not the spans, so `foldsInto` is unchanged; `appendOrFold` appends the folded entry's spans to the last element's own, on a freshly allocated slice, so it never writes through the input's backing array or its spans' backing array.
`cloneHistory` deep-copies `GateSpans` too, so `TestAppendOrFold_InputUnchanged` can see an aliased span list.

Tests:
- `gatedOutcomeProducer` takes the spans its pointer carries; `TestStep_GateAttempts`'s rows pass nil except a new row, "a gated call's spans persist", whose spans appear on the returned and the persisted entry; the ungated row reads the status file's raw bytes with `os.ReadFile`, decodes its first history entry into a `map[string]any` and asserts it has no `gate_spans` key, as an old entry reads, since the decoded `Status` cannot tell an absent list from an empty one.
- `TestAppendOrFold_Folds`: a row folding two equal-attempt exempt Stucks carrying one span each yields one entry with both spans in order.
- `TestAppendOrFold_InputUnchanged`: the gated input carries a span, and the fold of a spanned entry onto it leaves the input's spans untouched.
- `scriptedRerouteProducer`'s reroute pointer carries one span, and each of `Shed.rerouteStuck`'s three entries is asserted to carry it: the reroute entry in `TestStep_RerouteMovesToTargetAndEndsTargetSegmentEpisode`, the reroute-cap halt's capped Stuck in `TestRun_RerouteCapHaltsOnFourthRerouteAndResumeGrantsThreeMore` (beside its existing last-entry check), and the persisted Stuck of a row with no `OnReroute` in `TestStep_RerouteWithoutOnRerouteFailsRun`.
- `TestStatus_JSONRoundTrip` round-trips an entry with spans and decodes, strictly and through `ParseStatusTolerant`, an entry whose span carries an unknown key, keeping the span's known fields.

`doc.go` gains a sentence on the spans beside the gate count and on how a fold appends them.

**ImpactSummary:** additive fields on two shedengine records and their three writers; folding's match is unchanged, so no existing history folds differently; test helpers in the package only.

**Commit:** 3: shedengine: gate spans on history entries
