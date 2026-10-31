# Card 12 — operator-wait-rows

**Create:**
- `plan:internal/loomshed#OperatorWaitRows` -> `var OperatorWaitRows = []string{pawlengine.NameBeamAwait, NamePRReview, NameSamplerSession}`
- `plan:internal/loomshed#WaitsOnOperator` -> `func WaitsOnOperator(row string) bool`

**Edit:**
- `internal/battencli#childAtWaitingRow`
- `internal/battencli#readAgentActivity`
- `internal/battencli#TestReadAgentActivity_ReadsLiveRunsAndTheWaitMarker`
- `internal/loomcli#agentActivityFor` -> `func agentActivityFor(loc *lymcwd.Location, runID string, cfg shuttleengine.Config) func() (time.Time, bool, error)`
- `internal/loomcli#loomCLI.specFor`
- `internal/loomcli#TestAgentActivity_LiveLockWaitReadsAsActivity`
- `internal/loomshed#loomshedAllowedImports`

**Extends:**
- `internal/loomshed/doc.go`
- `internal/loomcli/doc.go`
- `docs/overview.md`

**Uses:**
- `internal/pawlengine#NameBeamAwait`
- `internal/loomshed#NamePRReview`
- `plan:internal/loomshed#NameSamplerSession`

**Intent:**
D7: one list of the rows a run's child may wait in with no agent working, `Beam-Await`, `PR-Review` and `Sampler-Session`, declared once in `loomshed`, which both `internal/battencli` and `internal/loomcli` import and which lies outside `internal/boardengine`.
`WaitsOnOperator` reports whether `row` is in `OperatorWaitRows`; loomshed's doc names the list and its two readers.
`childAtWaitingRow` keeps its status read and answers `loomshed.WaitsOnOperator(status.CurrentProducer)` for a running status, so dobby reads a child at Sampler-Session as a live wait and sends no quiet notice; its doc comment and `readAgentActivity`'s name the list instead of the two rows, and battencli drops its `pawlengine` import if nothing else uses it.
`agentActivityFor` gains `runID` and, after the existing readings and before the shuttle wait markers, reads the run's status file (`shedrun.StatusFile(loc, runID)` under its lock): a running status at a listed row reads as activity now, so the step watchdog never kills a step waiting on the operator; an absent or unreadable status reads as before, with no error.
`specFor` passes `c.runID`.
Bound: a wedged step at a listed row is no longer ended by the watchdog; only `lym batten cancel` or the row's own halts end it, which the doc comments say.
`TestReadAgentActivity_ReadsLiveRunsAndTheWaitMarker`'s status steps gain `RunningAtSamplerSessionIsAWait` and `BlockedAtSamplerSessionIsNoWait`, beside the existing Beam-Await, PR-Review and Parent-Align steps (D7 acceptance for dobby).
`TestAgentActivity_LiveLockWaitReadsAsActivity` passes the location's run id and gains rows writing the run's status file: running at Sampler-Session, at PR-Review and at Beam-Await each read activity now; running at Describe and blocked at Sampler-Session read none, and an unreadable status file (`{"current_producer":`) reads none with no error (D7 acceptance for the watchdog).
`loomshedAllowedImports` gains `internal/pawlengine` with a comment that it supplies a row-name constant and resolves no geometry, so `TestToldGeometryInvariant_AllowlistOnly` passes.
`loomcli/doc.go` and `docs/overview.md` gain the sentence that a step at a row waiting on the operator reads as active to the watchdog and as a live wait to dobby.

**ImpactSummary:** dobby's waiting-row reading and the watchdog's activity reading now share one list; `agentActivityFor` gains a run-id argument with one production caller and one test, and PR-Review and Beam-Await steps stop being killed after the idle window.

**Commit:** 12: one list of rows that wait on the operator, read by dobby and the step watchdog
