# Card 5 — Shed carries `parent_notice` on an `awaiting` halt

**Edit:**
- `internal/shedengine/producer.go`
- `internal/shedengine/status.go`
- `internal/shedengine/run.go`
- `internal/shedengine/doc.go`
- `internal/shedverbs/step.go`
- `contracts/specs/loom-status-spec.md`
- `internal/shedengine/awaiting_test.go`
- `internal/shedverbs/step_test.go`

**Intent:**
D5: an escalating Bouncer hands its parent a one-line notice, which Shed persists on the status file and the step envelope as an optional `parent_notice` field set only on `awaiting`.
Only stdlib is touched in `shedengine` (`PATTERN-shed-producer-seam`).

- `OutputPointer` gains `ParentNotice string`: advisory text Shed reads only on an `Awaiting` verdict, never for routing.
- `Status` gains `ParentNotice string` with `json:"parent_notice,omitempty"`.
- `StepResult` gains `ParentNotice`, set only alongside `StateAwaiting`.
- `persistTransient` writes the notice on every persist, as it writes `Transient`: the `Awaiting` arm passes the producer's notice normalized to one line by the same rule `stuckReason` applies, and every other write passes empty, so no later state carries a stale notice.
- `StepEnvelope` adds the `parent_notice` key only when `res.ParentNotice` is non-empty; its doc comment's closed key list names it as conditional.
- `loom-status-spec.md`'s schema gains the optional `parent_notice` line, and the `error` comment's description of `awaiting` names a review segment's escalation instead of only the circling halt.
- `doc.go` documents the field where it documents the `Awaiting` outcome.

Tests: an `Awaiting` verdict with a notice persists it and returns it on `StepResult`; an `Awaiting` without one and every other outcome leave the field absent from the JSON; a step after an awaiting-with-notice clears it; `StepEnvelope` carries `parent_notice` when set and lacks the key otherwise.

**ImpactSummary:** Additive fields on Shed's producer contract, status file and step envelope; no producer sets the notice until Card 10.

**Commit:** 5: Shed carries parent_notice on an awaiting halt
