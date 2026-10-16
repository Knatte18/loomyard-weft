# Card 7 — idle-probe-no-input-box

**Edit:**
- `internal/shuttleengine#IdleProbe`
- `internal/shuttleengine#Runner.SessionIdle`
- `internal/shuttleengine#TestRunner_SessionIdle_ReturnsScriptedClassification`
- `internal/shuttleengine/doc.go`

**Uses:**
- `internal/shuttleengine#readBox`
- `internal/shuttleengine#boxReading`
- `internal/shuttleengine#readinessReasonNoInputBox`
- `internal/shuttleengine#cyclerEngine`

**Intent:** First half of #686 (D7): `IdleProbe` reports a pane that shows no input box to an engine that reads one.

`IdleProbe` gains `NoInputBox`, true when `Idle` is false and the pane shows no input box to an engine with the `InputBoxReader` capability; always false when `Idle` is true, and false for an engine that reads no box.
`Runner.SessionIdle` sets it on the readiness path when the held reading's reason is `readinessReasonNoInputBox`, and on the pane-probe path, which a strand with no run record always takes, when the answer is not idle and `readBox` reports a reader that found no box.
No provider-specific check is added; the reading is the existing `InputBoxReader` one.
Callers that compare an `IdleProbe` or read its other fields are unchanged, since the field only adds information.

Test: in `TestRunner_SessionIdle_ReturnsScriptedClassification`, the rows "a pane with no readable box is held for an engine that reads one" and "idle-fresh with no readable box is held" want `NoInputBox` true beside their reason, and the subtest "a strand with no run record reads the pane for a draft" gains a row whose capture `no box here` wants `IdleProbe{NoInputBox: true}`, and a row over a `cyclerEngine`, which reads no input box, whose not-idle capture `no box here` wants `IdleProbe{}` with `NoInputBox` false; the existing idle and draft rows keep it false.

Docs in the same commit: the session-state paragraph of `internal/shuttleengine/doc.go` that lists what `Runner.SessionIdle` sets adds `NoInputBox`, on both the readiness reading and the pane fallback.

**ImpactSummary:** One field added to the exported `IdleProbe`, set in `Runner.SessionIdle` only; the orch watcher and creel adapter that read the probe compile unchanged, and only two rows and one subtest row of the classification test change.
