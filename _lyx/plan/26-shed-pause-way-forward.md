# Card 26 — shed-pause-way-forward

**Create:**
- `internal/shedengine#pauseCommandFor`

**Edit:**
- `internal/shedengine#Shed`
- `internal/shedengine#GotoRequest`
- `internal/shedengine#busyWayForward` -> `func busyWayForward(runID, pauseCommand string) string`
- `internal/shedengine#Goto`
- `internal/shedengine#Relaunch`
- `internal/shedengine#RenameProducers`
- `internal/shedengine#Shed.Run`
- `internal/shedengine#Shed.Step`
- `internal/shedbuild#ShedPaths`
- `internal/shedbuild#NewShedFrom`
- `internal/shedverbs#Spec`
- `internal/shedverbs#gotoCmd`
- `internal/battencli#battenCLI.wire`
- `internal/battencli#battenCLI.specFor`
- `internal/pairteardown#Teardown.busyError`
- `internal/loomcli#consumeArbiterAttempt`
- `internal/shedengine#TestGoto_RunLockHeldRefusesAndLeavesFile`
- `internal/shedengine#TestGoto_RunningRunRefusedUntilPaused`
- `internal/shedengine#TestRelaunch`
- `internal/shedengine#TestStep_ErrShedBusy`
- `internal/shedverbs#TestGotoCmd_RefusesHeldRunLock`
- `internal/shedverbs#TestGotoCmd_PassesToldWayForwardTexts`
- `internal/pairteardown#TestTeardown_CallSequence`
- `internal/loomcli#TestConsumeArbiterAttempt`
- `contracts/specs/refusal-spec.md`
- `internal/shedengine/doc.go`

**Intent:**
Point the shed-busy and goto refusals at the pause each recipe still has (D8): `pause --after <current_producer>` for a loom run and `lyx batten pause` for a batten run, and drop the arbiter-attempt refusal's requested-pause clause for loom.

- `Shed` and `GotoRequest` gain `PauseWayForward`, the told command a way forward names to stop the run at a boundary; `ShedPaths` carries it into `NewShedFrom`, and shedverbs' `Spec` into `gotoCmd`'s request.
- `pauseCommandFor(told, runID, producer)` returns the told command when one is set, and otherwise the shed default `"lyx shed pause <run-id> --after <producer>"` (the run-id omitted when none is told, `<current_producer>` when the producer is unknown).
- `busyWayForward(runID, pauseCommand)` names it: `way forward: <pauseCommand> halts the live driver at its next producer boundary; check the holder with "lyx shed status <run-id>", then retry`. `RenameProducers`, `Relaunch`, `Shed.Run`, `Shed.Step` and `Goto` pass `pauseCommandFor` of their told value; the busy path reads no status, so it names `<current_producer>`.
- `Goto`'s running-run refusal names `pauseCommandFor(req.PauseWayForward, runID, st.CurrentProducer)` with the run's real current producer, then `lyx shed step`, then goto again.
- Batten tells `"lyx batten pause <slug>"` through `battenCLI.wire`'s `ShedPaths` and `battenCLI.specFor`'s `Spec`; loom tells nothing and takes the `--after` default.
- `pairteardown`'s busy refusal for a live loop names `lyx shed pause <slug> --after <current_producer>`.
- `consumeArbiterAttempt` no longer refuses on `pause_requested`: a `pause_requested` left by an older binary is no longer refused here; Relaunch carries it through (shedengine keeps `pause_requested` on a re-launch) and the shed honours it at the next producer boundary, so the consumed attempt still halts there; a moved stop key or changed pause conditions still refuse.

`refusal-spec.md`'s `shed busy` and `goto on a running run` rows name the recipe's pause (`lyx shed pause <run-id> --after <current_producer>` for a loom run, `lyx batten pause <slug>` for a batten run), and the arbiter-attempt row drops "a requested pause".

Tests: `TestGoto_RunLockHeldRefusesAndLeavesFile`, `TestRelaunch`'s run-lock row and `TestStep_ErrShedBusy` assert the default names `lyx shed pause` with `--after <current_producer>`, and a told command replaces it; `TestGoto_RunningRunRefusedUntilPaused` asserts the refusal names the run's current producer after `--after`; `TestGotoCmd_RefusesHeldRunLock` and `TestGotoCmd_PassesToldWayForwardTexts` assert the default and that a told `lyx batten pause` text passes through the spec; `TestTeardown_CallSequence`'s held-loop row asserts the new wording; `TestConsumeArbiterAttempt`'s requested-pause row now consumes the attempt.
`doc.go`'s busy paragraph names the told command.

**ImpactSummary:** busyWayForward re-signed with its five callers in shedengine on this card; two shedengine request structs, ShedPaths and the shedverbs Spec gain one told string each, and batten is the one module that tells it.
