# Card 19 — raddle-checks-and-state

**Create:**
- `plan:internal/loomshed#RaddleMode` -> `type RaddleMode string`
- `plan:internal/loomshed#RaddleState` -> `type RaddleState struct`
- `plan:internal/loomshed#ReadRaddleState` -> `func ReadRaddleState(path, lockPath string) (RaddleState, error)`
- `plan:internal/loomshed#UpdateRaddleState` -> `func UpdateRaddleState(path, lockPath string, mutate func(*RaddleState)) (RaddleState, error)`
- `plan:internal/loomshed#RaddleCheck` -> `type RaddleCheck struct`
- `plan:internal/loomshed#CheckRaddleProposal` -> `func CheckRaddleProposal(check RaddleCheck, filing bool) ([]raddleparser.Finding, error)`
- `plan:internal/loomshed#RecheckRaddleBoard` -> `func RecheckRaddleBoard(check RaddleCheck) (raddleparser.Recheck, error)`
- `plan:internal/loomshed#NewRaddleGate` -> `func NewRaddleGate(check RaddleCheck, onPass func() error) shuttleengine.Gate`
- `plan:internal/loomshed#StoreColdProposal` -> `func StoreColdProposal(proposalPath, coldPath, statePath, lockPath string) error`
- `plan:internal/loomshed#TestRaddleState` -> `func TestRaddleState(t *testing.T)`
- `plan:internal/loomshed#TestCheckRaddleProposal` -> `func TestCheckRaddleProposal(t *testing.T)`
- `plan:internal/loomshed#TestNewRaddleGate` -> `func TestNewRaddleGate(t *testing.T)`

**Extends:**
- `internal/loomshed/doc.go`

**Uses:**
- `plan:internal/raddleparser#Parse`
- `plan:internal/raddleparser#Check`
- `plan:internal/raddleparser#RecheckBoard`
- `plan:internal/raddleparser#BoardContext`
- `internal/loomshed#NewDiscussionGate`
- `internal/loomshed#DiscussionCheck`
- `internal/shuttleengine#Gate`
- `internal/shuttleengine#GateResult`
- `internal/state#UpdateJSON`
- `internal/state#ReadJSON`

**Intent:** The raddle run's state and the one place D6's checks and D8's re-check run against a proposal on disk, shared by the gate, the two verbs and Raddle-File.
`RaddleMode` is `auto` or `interactive`.
`RaddleState`, the run's `state.json` (card 9's path, read and written through `internal/state`, tolerant of unknown keys), holds: `mode` (empty until Raddle-Write's first spawn records it, D7); `cold_stored`; `session`, the number of the current Raddle-Write session, which Raddle-Write raises at every call of its row, so a resume after a session died, a halt's resume and a goto each count as a new session (card 21); `proposed_session`, the session of the last passing `raddle-propose`; `notice_session`, the session whose waiting notice went out; `approval` (`{hash, session}`, cleared by a return); `returns`, Raddle-File's count of returns; `return_reason`, the reason of a return not yet taken up by a new session; and `notice_sent`, Raddle-File's step 6 marker.
`RaddleCheck` carries the proposal path, the decision record path, and `Board func(filing bool) (raddleparser.BoardContext, error)`, built by the caller from the board, the raddle store and the profiles.
`CheckRaddleProposal` reads and parses the proposal (a parse error is one finding `proposal-unparsable` carrying the parser's way forward), reads the record and runs `raddleparser.Check` over the context; a read error of the record or the board is returned.
`RecheckRaddleBoard` runs `raddleparser.RecheckBoard` over the current proposal and context.
`NewRaddleGate` is the `raddle` gate, shaped like `NewDiscussionGate`: it runs `CheckRaddleProposal(check, false)`, fails with the joined findings as `GateResult.Findings`, and on a pass calls `onPass` (Raddle-Write's auto mode stores the cold proposal there) before passing; an `onPass` error is an infrastructure error, which burns no attempt.
`StoreColdProposal` copies the proposal to the cold path and sets `cold_stored` in one state update, and does nothing once `cold_stored` is set, so the cold proposal is stored once per run and never overwritten; the propose verb and the auto-mode gate both store through it.
`TestRaddleState` (tier0) round-trips the state and keeps an unknown key, and covers `StoreColdProposal`: a first call stores the proposal, a second call after the proposal changed leaves the cold file as it was.
`TestCheckRaddleProposal` (tier0, a fixture proposal and record in a temp dir, a stub `Board`) has rows: a valid proposal (no finding); one D6 finding passing through; an unparsable file; a record read error (returned).
`TestNewRaddleGate` (tier0) has rows: pass (onPass called once), fail (findings, onPass not called), onPass error (returned as error).
`doc.go` gains a raddle paragraph.

**ImpactSummary:** New loomshed members only; nothing reads them until the verbs and the raddle rows.
