# Card 6 — prrework-carry-gate

**Edit:**
- `internal/loomshed#prRework.Call`
- `internal/loomshed#prRework.commitRound` -> `func (p *prRework) commitRound(gated shedengine.OutputPointer) (shedengine.Outcome, shedengine.OutputPointer, error)`
- `internal/loomshed#prRework.finish` -> `func (p *prRework) finish(clear bool, gated shedengine.OutputPointer) (shedengine.Outcome, shedengine.OutputPointer, error)`
- `internal/loomshed#reworkFixture`
- `internal/loomshed#reworkInner.Call`
- `internal/loomshed#TestPRRework_ClassRecorded`

**Intent:**
PR-Rework's session runs gated on `rework-plan` through `MultiLLMProducer`, which card 5 makes carry the gate's count and spans, but `prRework.Call` drops that pointer on two exits: its Done builds a fresh pointer in `finish`, and its "rework plan does not parse" Stuck builds one in `stuck` (D5: every gated row copies its run's count and spans, whether or not it copies a count today).
Both exits now keep the session pointer's `GateAttempts` and `GateSpans`.

`commitRound` and `finish` take the gated pointer whose gate fields the Done pointer carries; `finish` copies only `GateAttempts` and `GateSpans` from it onto the Done pointer it returns, so the Done pointer's `Path` stays empty as today.
The callers pass what they hold: `prRework.Call` passes the session's pointer after the session ran, and passes the zero pointer on the three paths where no session ran this call: the crash-resume path that finds the round already classified (`commitRound`), the absent-record path (`finish(false, ...)`) and the re-entry whose rejection a committed round already covers (`finish(true, ...)`), whose history entries stay ungated, as today.
The parse-failure Stuck sets the same two fields from the session pointer on the pointer `stuck` builds.
A Stuck or error the session itself returns already passes its pointer through unchanged.

Tests:
- `reworkFixture` gains `gateAttempts *int` and `gateSpans []shedengine.GateSpan`, which `reworkInner.Call` stamps on the Done pointer it returns.
- `TestPRRework_ClassRecorded`'s "mixed" row sets both and asserts the producer's returned Done pointer carries them; the "all prosa" row leaves them unset and asserts the pointer carries neither.
  `reworkFixture.call` discards the pointer, so the row calls `f.producer().Call` itself.
- `TestPRRework_ClassRecorded` gains a row whose fake session (`onInner`) leaves a plan that `planparser.ParsePlan` refuses, with `gateAttempts` and `gateSpans` set on the fixture; the row calls `f.producer().Call` and asserts the outcome is Stuck, the reason starts with `rework plan does not parse`, and the pointer carries the fixture's gate count and spans.

**ImpactSummary:** one producer's exit pointers in `internal/loomshed/prrework.go`, two private re-signed helpers whose callers are all in `prRework.Call`; routing is unchanged, since shed only records the gate fields.

**Commit:** 6: loomshed: PR-Rework keeps its session's gate count and spans
