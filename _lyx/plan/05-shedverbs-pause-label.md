# Card 5 — shedverbs-pause-label

**Edit:**
- `internal/shedverbs#pauseFlags`
- `internal/shedverbs#applyPauseConditions`
- `internal/shedverbs#pauseCmd`
- `internal/shedverbs#statusCmd`
- `internal/shedverbs#Hooks`
- `internal/shedverbs#Spec`
- `internal/shedverbs#RunGoto`
- `internal/shedverbs#RenderStatusHuman`
- `internal/shedverbs#writeActivityTail`
- `internal/shedverbs#TestPauseCmd_SetsPauseRequested`
- `internal/shedverbs#TestPauseCmd_Conditions`
- `internal/shedverbs#TestStatusCmd_FoundEnvelope_GenericCore`
- `internal/shedverbs#TestRenderStatusHuman`
- `internal/shedverbs#TestGotoCmd_PassesToldWayForwardTexts`
- `internal/shedverbs/doc.go`
- `internal/loomcli#loomVerbTexts`
- `internal/loomcli#TestStatusCmd_EnvelopeKeySet`
- `internal/shedcli#shedVerbTexts`
- `internal/battencli#battenVerbTexts`

**Intent:** D4 in the verbs, and the D2 seam's passthrough.

The pause verb gains a visible `--by <label>` flag, a free-text caller label such as `operator` or the orch's strand name.
It gates or skips nothing: it is stored beside the request it accompanies, `PauseRequestedBy` for a bare pause, `PauseBeforeBy` with `--before` and `PauseAfterBy` with `--after`, replacing a recorded label with its condition; `--clear` clears both condition labels with the conditions.
`pauseFlags` gains `by`, and `applyPauseConditions` applies the labels.
`--by shed` is refused before any write, with `pause --by does not take shed, the label of lyx's own automatic steps; way forward: re-run pause with your own label, such as --by operator`; the refusal is reached by a pause with `--by shed`.
A label without a request (bare `--by` under loom's `BarePause` refusal) writes nothing, since the hook still refuses first.

The status envelope's generic core gains `pause_reason`, the status file's value, and `Hooks.StatusExtras`'s doc names the eleven-key core.
`RenderStatusHuman` prints, through `writeActivityTail`, a `why <pause_reason>` line for a paused run with a non-empty reason.

`Spec` gains `GotoGuard shedengine.GotoGuard`, told by the arming module, and `RunGoto` passes it on `GotoRequest.Guard`; nil admits as today.

The three pause help texts (`loomVerbTexts`, `shedVerbTexts`, `battenVerbTexts`) name `--by` and that the label is recorded in the paused run's `pause_reason`, and the status help texts of `loomVerbTexts` and `battenVerbTexts`, which enumerate the envelope's keys, name `pause_reason` among them; `shedVerbTexts`' status text defers to the recipe's own status verb and names no key, so it is unchanged; no test pins the prose.
`doc.go` states the label and the new core key.

Tests: `TestPauseCmd_SetsPauseRequested` adds a bare pause with `--by operator` storing `pause_requested_by`, and a bare `--by operator` under a refusing `BarePause` hook that leaves the file unchanged; `TestPauseCmd_Conditions` adds rows for `--before X --by orch`, `--after X --by orch`, `--clear` dropping both labels, the `--by shed` refusal leaving the file unchanged, and a `--before X` over a status seeded paused with `pause_reason` `pause requested by operator`, whose write keeps that reason byte for byte; `TestStatusCmd_FoundEnvelope_GenericCore` asserts the key set includes `pause_reason` with the file's value; loom's `TestStatusCmd_EnvelopeKeySet`, which refuses every key outside its `wantKeys` set, adds `pause_reason` to that set; `TestRenderStatusHuman` adds a paused row with and without a reason; `TestGotoCmd_PassesToldWayForwardTexts` asserts a told `GotoGuard` reaches the request, its refusal leaving the file unchanged.

**ImpactSummary:** Additive flag and envelope key on the generic verbs used by loom, shed and batten; help texts only in the three CLI modules.

**Commit:** 5: shedverbs: pause takes a caller label, status shows the pause reason, goto passes the told guard
