# Card 2 — shared-hub-board

**Create:**
- `plan:internal/boardengine#OpenHub` -> `func OpenHub(hubPath string) (*Board, error)`
- `plan:internal/boardengine#Board.SetRunStatus` -> `func (b *Board) SetRunStatus(slug, status string) error`

**Edit:**
- `internal/loomcli#loomCommitStatusDeps`
- `internal/loomcli#landingDeps`
- `internal/loomcli#boardStatus`
- `internal/boardengine/board_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Delete:**
- `internal/loomcli#openHubBoard`

**Intent:**
One opener and one run-status writer for the hub's board, so loom and batten's claim (Card 3) share them rather than keeping two copies.

- `OpenHub(hubPath)`, in `internal/boardengine/config.go`, is `openHubBoard`'s body over a told hub path: `LoadConfig(fabricengine.BoardDir(hubPath), "board")`, `Path` set to that directory, then `New(ApplySkipEnv(cfg))`.
  It reaches `_board` from the hub path only, never through a worktree link (`PATTERN-hub-containment`), and derives no cwd (`PATTERN-told-geometry`).
- `Board.SetRunStatus(slug, status)`, in `board.go`, sets `status` on slug's entry under the write lock, and returns a `noWrite` with nil error when the entry is absent or `done`: the skip loom's `SetBoardStatus` closure does today, moved under the lock.
- `loomCommitStatusDeps`'s `SetBoardStatus` becomes `boardengine.OpenHub(location.HubPath)` then `SetRunStatus(shedrun.ResolveRunID(location, runID), status)`.
- `landingDeps`'s `MarkTaskDone` opens the board with `boardengine.OpenHub(l.HubPath)`; it keeps its plain `SetStatus` with `done`, which errors on an absent slug as today.
- `boardStatus` composes through `boardengine.RunStatus(state, producer)`; it still writes nothing for a done run, so loom writes the same statuses as before.
- `openHubBoard` is deleted once both callers use `OpenHub`.

The existing `landingdeps_markdone_test.go` and `wiring_commitstatus_test.go` cover the opener and the status form unchanged.
New behavior is `SetRunStatus`'s skip: add one table-driven `TestSetRunStatus` to `board_test.go` with rows for an entry with no status (set), a `done` entry (left `done`, no write), an absent slug (nil error, no entry created) and a locked entry (status replaced, which the Card 1 lock lets through).
Raise `internal/boardengine`'s row in `test-budget.yaml` by one.

**ImpactSummary:** loom's per-transition board write and landing's done write open the board through `OpenHub`; observable statuses and dispositions are unchanged.

**Commit:** 2: board: share the hub-board opener and the run-status write with loom
