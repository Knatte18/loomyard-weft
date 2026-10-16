# Card 8 — no-input-box-stall

**Edit:**
- `internal/creelengine#Stall.Reason`
- `internal/creelengine#Idleness`
- `internal/creelengine_test#TestScheduler_Injects`
- `internal/creelcli#hubDelivery.Idle`
- `internal/creelcli#stubOps`
- `internal/creelcli#stubOps.SessionIdle`
- `internal/creelcli#TestHubDelivery`
- `internal/creelengine/doc.go`

**Create:**
- `plan:internal/creelengine#StallNoInputBox` -> `const StallNoInputBox Stall = "no-input-box"`

**Uses:**
- `internal/shuttleengine#IdleProbe`
- `internal/creelengine_test#awaitOutcome`
- `internal/creelengine_test#env.driveProbeWaits`
- `internal/creelengine_test#stalledReasons`
- `internal/creelengine_test#newEnv`

**Intent:** Second half of #686 (D7): a pane with no input box names a stall.

`StallNoInputBox` is the stall `no-input-box`, and `Stall.Reason` renders it as: the pane shows no input box: it runs no agent, or a dialog covers the box, and the delivery waits until the box shows.
`hubDelivery.Idle` maps a not-idle probe's `NoInputBox` to it, after too-short and draft in precedence, and a not-idle probe with none of the three still names no stall.
creelengine reads the field only through the adapter's `Idleness`, never `IdleProbe` itself (`PATTERN-creel-store`).
Bound: the stall only names a reason; it changes no idle answer and types nothing.

Tests:
- `TestHubDelivery`'s subtest "Idle reports the stub's probe": `stubOps` gains `noInputBox`, its `SessionIdle` answers it, and rows pin no input box alone giving `StallNoInputBox`, and too-short with no input box and draft with no input box each keeping the stronger stall.
- `TestScheduler_Injects`'s existing row "InjectStalledForItsWholeTTLStillExpiresWithOneStalledRecord" scripts alpha's stall as `StallNoInputBox` in place of the draft and is driven probe by probe until it is answered, in place of one two-minute advance; it asserts the outcome expired with stall `no-input-box`, the `expired` log entry's reason `no-input-box`, and one `stalled` record naming it, so the acceptance criterion's end-to-end value is pinned with no new row.

The doc comment of the `Stall` field of `Idleness` stops naming a dialog among the not-idle readings that name no reason, since a dialog that covers the input box now names `no-input-box`.

Docs in the same commit: the Layering paragraph of `internal/creelengine/doc.go` lists a pane with no input box beside a pane too short and a draft among the stalls the idle answer names.

**ImpactSummary:** One new exported stall value in `internal/creelengine` and one more case in the creel adapter's probe mapping; the stall journal, the waiters and the expiry treat it like the existing stalls, and no signature changes.
