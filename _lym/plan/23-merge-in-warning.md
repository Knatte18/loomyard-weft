# Card 23 — merge-in-warning

**Create:**
- `internal/weaverengine#RunInFlight`
- `internal/weaverengine_test#TestRunInFlight`
- `internal/fabriccli_test#seedWeaverState`

**Edit:**
- `internal/fabriccli#websterInFlightWarnings`
- `internal/fabriccli#websterInFlightWarning`
- `internal/fabriccli_test#TestCommand_MergeScenario`
- `internal/fabriccli/doc.go`

**Extends:**
- `internal/weaverengine/doc.go`

**Uses:**
- `internal/weaverengine#ActiveOrSetting`
- `internal/weaverengine#outcomeRecord`
- `internal/websterengine#RunInFlight`
- `internal/fabriccli_test#seedWebsterState`

**Intent:** `lym fabric merge-in` warns of the active implementer's run in flight (D9, D13).

- `weaverengine.RunInFlight` reports a Weaver run in flight while its state file exists and its `outcome.yaml` is absent or not done, as `websterengine.RunInFlight` does for Webster.
  It relies on `Run` removing `outcome.yaml` on entry (card 12), so a re-stepped run reads as in flight until its Done return.
  It decodes into `outcomeRecord`, refusing an unknown key: `done` is not in flight, `stuck` is in flight, and any other `outcome` value or an undecodable file is the error its `TestRunInFlight` row names.
- `websterInFlightWarnings` asks the seam's advisory form, `ActiveOrSetting`, with the setting read from the location's config dirs, and checks the active implementer's in-flight read; a seam refusal falls back to the setting and is logged, so a corrupt Webster state never blocks the warning.
  The warning text names the active implementer: Webster's keeps its record-batch clause, Weaver's says a batch strand's commits sit on the task branch and its post-batch checks run against them.
- `fabriccli/doc.go`'s merge-in paragraph names both implementers.

`TestRunInFlight` is tier0: no state, a state with no outcome, a stuck and a done outcome, and an unreadable outcome.
`TestCommand_MergeScenario` gains steps after Webster's, through `seedWeaverState`: a Weaver run with no outcome warns, and one whose `outcome.yaml` reads done does not; with `tapestry.implementer` set to `weaver`, a Weaver run with no outcome beside a corrupt Webster `state.json` still warns of Weaver's run.

**ImpactSummary:** One fabriccli warning reads the seam; its error and envelope shapes are unchanged.
