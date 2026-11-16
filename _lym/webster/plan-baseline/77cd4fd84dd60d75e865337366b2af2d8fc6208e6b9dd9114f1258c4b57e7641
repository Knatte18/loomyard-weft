# Card 10 — board-done-pushed

**Create:**
- `plan:internal/boardengine#Board.SetStatusPushed` -> `func (b *Board) SetStatusPushed(slug string, status *string) error`
- `plan:internal/boardengine#TestBoard_SetStatusPushed` -> `func TestBoard_SetStatusPushed(t *testing.T)`

**Edit:**
- `internal/boardengine#pushOwnerCommit`
- `internal/boardengine/doc.go`
- `internal/loomcli#landingDeps`
- `internal/loomcli#TestLandingDeps_MarkTaskDone_SetsStatusDone`

**Uses:**
- `internal/boardengine#SetOwners`
- `internal/loomcli/doc.go`

**Intent:**
A Bolt write that carries a decision pushes synchronously before its writer goes on (D1).
The run claim and the owner already do: `SetOwners` pulls first, commits and pushes through `pushOwnerCommit`, and batten's claim goes through it.
The landing's board `done` does not: `Board.SetStatus` spawns the detached sync.

`Board.SetStatusPushed` writes slug's status the way `SetOwners` writes: it pulls the Bolt first through `PullThenCommitWritten` with the status write as its write, and pushes through `pushOwnerCommit`, waiting for the push;
it spawns no detached sync.
Under `BOARD_SKIP_GIT` or `BOARD_SKIP_PUSH` it only commits, as `SetOwners` does, so under the push skip the board `done` follows the local merge (D2).
An unreachable origin returns an error wrapping `ErrOwnerOriginUnreachable`.
`pushOwnerCommit`'s doc and failure text drop the owner-only wording, since it now pushes a status too.
The board doc's write-path section says which writes push synchronously (claim, owner, a landing's `done`) and that a display status stays on the detached sync.

The landing's `MarkTaskDone` in `landingDeps` calls `SetStatusPushed(slug, done)`.
Card 11 moves the call after the parent push.

Tests:
- `TestBoard_SetStatusPushed` (tier1, board fixture with an origin): origin's Bolt holds the status once the call returns, which the detached sync, a no-op in a test binary (`SpawnSync`), never makes true; a row with the origin unreachable returns `ErrOwnerOriginUnreachable` with the commit local; a skip-push row commits only.
- `TestLandingDeps_MarkTaskDone_SetsStatusDone` (tier0) keeps its assertion under the skip env it runs in, now through the pushed write.

**ImpactSummary:** Adds one synchronous board status writer and moves the landing's done mark onto it; `SetStatus` and every display-status writer keep the detached sync.
