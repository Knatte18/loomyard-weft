# Card 2 — shuttle-gate-spans

**Create:**
- `plan:internal/shuttleengine#GateSpanKind` -> `type GateSpanKind string`
- `plan:internal/shuttleengine#GateSpanGate` -> `const GateSpanGate GateSpanKind = "gate"`
- `plan:internal/shuttleengine#GateSpanHeld` -> `const GateSpanHeld GateSpanKind = "held"`
- `plan:internal/shuttleengine#GateSpanRework` -> `const GateSpanRework GateSpanKind = "rework"`
- `plan:internal/shuttleengine#GateSpan` -> `type GateSpan struct`
- `plan:internal/shuttleengine#Run.endReworkSpans` -> `func (run *Run) endReworkSpans(at time.Time)`
- `plan:internal/shuttleengine#Run.closeGateSpans` -> `func (run *Run) closeGateSpans(at time.Time)`
- `plan:internal/shuttleengine#spanKinds` -> `func spanKinds(spans []GateSpan) []GateSpanKind`

**Edit:**
- `internal/shuttleengine#GateEntryOutcome`
- `internal/shuttleengine#Run`
- `internal/shuttleengine#Run.Wait`
- `internal/shuttleengine#Run.handleGatedBoundary`
- `internal/shuttleengine#Run.evaluateGate`
- `internal/shuttleengine#Run.finalize`
- `internal/shuttleengine#TestGate_FirstDoneSettlesWithoutReprompt`
- `internal/shuttleengine#TestGateList_FinalReports`
- `internal/shuttleengine#TestGatePending_HoldsSendsOnceAndPasses`
- `internal/shuttleengine#TestGate_DeadlineExpiresBetweenAttempts`
- `internal/shuttleengine#TestGate_SendFailsMidLoop_EndsLoopWithAttemptsSoFar`
- `internal/shuttleengine#TestGate_UnsentRepromptRetries`
- `internal/shuttleengine#TestGatePending_MidTurnTicksNeverReevaluate_DeadlineUsesFinal`
- `internal/shuttleengine#TestAttach_GateThreading`
- `internal/shuttleengine/doc.go`

**Intent:**
Shuttle's gate outcome reports, per gate entry, the spans that entry took on the run clock (D5).
`GateSpan` (in `gate.go`) holds `Kind GateSpanKind`, `Start time.Time` and `End time.Time`; the kind is the closed set `GateSpanGate`, `GateSpanHeld` and `GateSpanRework`.
`GateEntryOutcome` gains `Spans []GateSpan`, in the order the spans started.
The spans are measured in the provider-neutral wait loop only (PATTERN-shuttle-provider-seam), from `run.clock`, so a test drives them with the package's fake clocks (PATTERN-test-speed).

`Run` gains three in-memory fields, index-aligned with `gate` and sized where `gateFails` is: `gateSpans [][]GateSpan`, the closed spans of each entry; `gateReworkSince []time.Time`, the start of each entry's open rework span, zero when none; and `gateHeldAt int` with `gateHeldSince time.Time`, the entry whose held span is open and its start, `gateHeldAt` -1 when none is open.
They live in `Wait`'s memory like `gateFails`, so a run `AttachGated` rebuilds after a driver restart starts them afresh, as it starts the attempt count.

The spans are recorded as follows.
- **gate:** `evaluateGate` reads the clock before and after each closure it runs, beside `beginGateWait`/`endGateWait`, and appends a gate span to that entry, but only when no held span is open at the start of the evaluation.
  While an entry is held pending, the re-evaluations record no gate span for any entry of the gate, so an entry listed before the held one gains no span per recheck and no two spans of one entry overlap.
- **held:** at the end of `evaluateGate`, an evaluation that stopped at a pending entry with no held span open opens one for that entry at the evaluation's end, after the gate span of the evaluation that first returned pending; an evaluation that did not stop pending at the held entry closes its held span at the evaluation's end.
  Both rules run in the finalize evaluation too, so the closure `Final` reads is covered by the hold.
- **rework:** `handleGatedBoundary` opens a rework span for the failed entry at the clock reading after `sendWithin` delivered its re-prompt, beside `gateSent[failed]++`; a send that fails, busy, unlanded or otherwise, opens none.
  `Wait`'s gated-Done branch (`outcome == OutcomeDone` with a gate) calls `endReworkSpans` with the arrival's clock reading before it sets `gateAtBoundary`, which closes every open rework span at the writer's next arrival.
- **finalize:** `finalize`, after its `evaluateGate(true)` and only when `result.Gate` is non-nil, calls `closeGateSpans` with `result.EndedAt`, which ends every span still open (a rework span after a deadline, a lost session, a failed send or a terminal failure; a held span at a deadline) at finalize, and then stamps a copy of each entry's spans onto a copy of the outcome's `Entries`, so the memoised verdict is never written through.
  The outcome carries the spans on every result that carries a gate, done, failed or session-lost alike; a non-done outcome (died, timeout without the output files) still carries no `Gate`, as today.
  That is the bound of D5's "every result": every carrier maps a died or timed-out shuttle result to a returned error, which writes no history entry, so no record could hold those spans.
- A span whose end reads before its start is kept as recorded; the checklist draws it as zero (card 10).

Tests, all on the fake clocks and existing fixtures, with each `//testtiming:keep` reason updated to name the spans it now pins:
- `TestGate_FirstDoneSettlesWithoutReprompt`: the gated row's closure advances the fake clock by three seconds (`fc.asleep`); the entry reports zero attempts and exactly one gate span three seconds long, starting at the arrival's clock reading; and after `Wait` returns, the run's memoised verdict (`run.gateVerdict`) still holds no spans, so finalize stamped a copy.
- `TestGateList_FinalReports`: the "exhaustion without pass_on_cap" row, where entry `b` is refused twice, asserts through `spanKinds` that `b`'s spans are gate, rework, gate, rework, gate, that each rework span starts at its re-prompt's delivery and ends at the next arrival's clock reading, and that `a`'s spans are three gate spans; no two spans of one entry overlap.
- `TestGatePending_HoldsSendsOnceAndPasses`: the spec gains a first entry `a` that always passes, before `parent-review`, and `wantStates` names both entries passed; `a` reports exactly one gate span (the evaluation before the hold opened) and none for the two evaluations run during the hold; `parent-review` reports one gate span then one held span, which ends at the evaluation that passed and starts no earlier than the gate span's end.
- `TestGate_DeadlineExpiresBetweenAttempts` becomes a table of two rows over the same one-re-prompt gate and captures, each row with its own config, deadline and status queue:
  the deadline expiring after the re-prompt keeps today's config (`LivenessEveryNPolls: 1_000_000`), its 2ms deadline and `liveStrandStatus(true)`;
  the session lost after the re-prompt takes `TestGate_NoLiveSessionDonePaths`'s `liveness` config (`LivenessEveryNPolls: 1`) with a deadline an hour out, and a status queue that answers live for every status read the boundary and the send make and then not-tracked, whose last entry repeats, so `maxStatusRetries` consecutive not-tracked liveness ticks end the run through `finishedDespiteMechanismFailure`'s Done, the output file being present.
  Each row asserts the entry's spans are gate, rework, gate with exactly one re-prompt sent, and that the rework span ends at `result.EndedAt`, closed at finalize; the deadline row keeps today's attempt and closure-call assertions, and the session-lost row asserts the run reached `EndedAt` before the deadline, so it did not end through the deadline branch.
- `TestGate_SendFailsMidLoop_EndsLoopWithAttemptsSoFar` (a send that fails outright) and `TestGate_UnsentRepromptRetries` (a send that fails busy or unlanded before one lands) assert that a failed send opens no rework span: the first's entry reports gate spans only, and the second's reports exactly one rework span, opened at the delivery that landed.
- `TestGatePending_MidTurnTicksNeverReevaluate_DeadlineUsesFinal`, whose run ends at the deadline while its entry is held, asserts the entry's spans are one gate span then one held span ending at `result.EndedAt`, and that the finalize evaluation, which runs `Final`, adds no gate span.
- `TestAttach_GateThreading`'s gated rows assert the attached run's entry reports only the gate span of its own evaluation, so spans start afresh on attach.
- `spanKinds` is a test helper in `gate_test.go` returning the kinds of spans in order.

`gate.go`'s file header, which lists the gate contract's exported types, names `GateSpan` and `GateSpanKind`.
`doc.go`'s gate passage gains the spans: the three kinds, the no-span-during-hold rule, the close at finalize and the reset on attach.

**ImpactSummary:** shuttle-internal: one new field on `GateEntryOutcome` that every existing reader ignores, run-local bookkeeping in the wait loop, and existing gate tests extended; no exported signature changes.

**Commit:** 2: shuttle: per-entry gate spans in the gate outcome
