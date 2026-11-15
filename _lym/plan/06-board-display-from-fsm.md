# Card 6 — board-display-from-fsm

**Edit:**
- `internal/battencli#newCommitStatusSeam`
- `internal/battencli#battenCLI.claimBoardEntry`
- `internal/battencli#TestNewCommitStatusSeam_WritesBoardStatusBeforeRunShed`
- `internal/battencli#TestWire_CreateWorktreeClaimsTheBoardEntry`
- `internal/loomcli#newCommitStatusSeam`
- `internal/loomcli#TestNewCommitStatusSeam_BoardStatus`

**Uses:**
- `plan:internal/runfsm#TransitionState`
- `internal/loomcli/doc.go`

**Intent:** The board's `<state> · <row>` status becomes a display written from the FSM state (D1), not from the shed state.
Every writer composes it with `boardengine.RunStatus(string(runfsm.TransitionState(...)), producer)`:
- batten's seam, which writes only at rows before `Run-Shed`, passes started and landing false, so a running transition reads `starting · <row>`;
- the claim `claimBoardEntry` writes reads `starting · Worktree-Create` the same way;
- loom's seam passes started true and landing true at `loomshed.NameFinalize` or a `done` state, so the child reads `running`, `landing`, `paused` or `halted`.
No decision reads the state half: `RunHolds`, `ProjectRecordHeld` and the run lock read the producer half, and `IsRunStatus` only the shape.
The board render's `atCell` shows the producer alone for a `running` status and the whole status otherwise, so a `starting`, `landing`, `paused` or `halted` row shows its state beside the row; that is the display D1 asks for.
The pawl and raddle writers already write `running`, which is the FSM's own word.

Tests, tier0: the two board-status tests gain rows asserting the written text for a running, a halted and a paused transition, and, for loom, a Finalize one.
`TestWire_CreateWorktreeClaimsTheBoardEntry` (tier0) asserts the claim `starting · Worktree-Create` where it asserts `running · Worktree-Create` today.

**ImpactSummary:** Three board writers change the state word they write; readers parse only the producer half, so no reader changes, and the board render shows the new words.
