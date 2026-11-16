# Card 13 — status-shows-owed

**Edit:**
- `internal/battencli#RunCells`
- `internal/battencli#DobbyRunCells`
- `internal/battencli#NewDobbyRunCells`
- `internal/battencli#DobbyRunCells.Of`
- `internal/battencli#DobbyLines`
- `internal/battencli#TestDobbyLines`
- `internal/battencli#TestDobbyRunCells_ReadsTheRunsFromDisk`
- `internal/battencli#TestDobbyRunCells_ResolvesTheChildPairAgainInEachRunShedSpell`

**Uses:**
- `internal/beamcli#dobbyStatus`
- `internal/beamcli/doc.go`
- `internal/pairteardown#ReadRun`

**Intent:**
`lym beam status` shows the owed push and its last error, and a missing code tip with the machine that owes it (D1, D2).
`RunCells` gains `Owes string`.
`DobbyRunCells` gains a verdict seam, `verdict func(slug string) (runfsm.Verdict, error)`, which `NewDobbyRunCells` fills with `pairteardown.ReadRun` and `runfsm.Judge` with nothing set aside.
`Of` sets `Owes` from it:
`push owed: <PushOwedError>` for a `push_owed` verdict (the bare `push owed` when no error is recorded),
`code tip <first 12 of CodeTip> missing on origin, owed by <CodeTipMachine>` for a `code_tip` one, and empty otherwise.
An unreadable verdict leaves `Owes` empty and logs at Debug, as the other cells' reads do.
`DobbyLines` appends a non-empty `Owes` to the run's pass cell.
`beamcli`'s `dobbyStatus` builds the cells through `NewDobbyRunCells` already, so it needs no change.
The failed-push record and its creel notice stay as they are.

Tests:
- `TestDobbyLines` (tier0): rows for a run with each `Owes` text asserting the rendered line.
- `TestDobbyRunCells_ReadsTheRunsFromDisk` sets the verdict seam: a `push_owed` verdict with an error and a `code_tip` verdict fill `Owes`, and a verdict read error leaves `Owes` empty.
- `TestDobbyRunCells_ResolvesTheChildPairAgainInEachRunShedSpell` (tier0) sets the verdict seam to a fake answering act, so no tier0 test runs the real reader.

**ImpactSummary:** One cell added to the dobby status lines, read through the act rule's existing reader; `lym beam status` and batten's overview are its only renderers.
