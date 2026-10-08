# Card 3 — batten-claim

**Edit:**
- `internal/battencli#battenCLI.wire`
- `internal/battencli/cli.go`
- `internal/battencli/wire_seams_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/boardengine#OpenHub`
- `plan:internal/boardengine#Board.SetRunStatus`
- `plan:internal/boardengine#RunStatus`

**Intent:**
`lyx batten run <slug>` claims the board entry before the pair exists, so the run lock starts at the claim instead of at loom's first transition.

In `wire`, the `CreateWorktree` closure's first statement, before the fabric config load, the already-complete probe and `Topology.Add`, writes the claim: `boardengine.OpenHub(location.HubPath)` then `SetRunStatus(slug, boardengine.RunStatus(string(shedengine.StateRunning), battenrecipe.NameWorktreeCreate))`, giving `running · Worktree-Create`.
An absent or `done` entry is skipped by `SetRunStatus`.
A failed open or write logs a `logger.Warn` naming the slug and the error, and the row continues: the board is the hub's overview, never the run's bookkeeping.
The claim stays when the row then fails (`Topology.Add` refuses, or the pair is incomplete); the stopped run resumes with `lyx batten run <slug>`.
Batten writes no board status for Seed-Child, Run-Shed or Worktree-Teardown; loom's transitions are the only later writer.
The claim lives in `battencli`'s wiring closure, never in `battenshed` or `shedengine` (`PATTERN-shed-producer-seam`, `PATTERN-told-geometry`).

What this lets through, bounded: a failed claim write leaves the lock window open until loom's first transition, as today.

Test: one `TestWire_CreateWorktreeClaimsTheBoardEntry` in `wire_seams_test.go`, table-driven over a wired prime whose temp hub holds only the board config (written as `landingdeps_markdone_test.go` does), so `CreateWorktree` claims and then fails on the absent fabric config before any pair is created.
Rows: an entry with no status reads `running · Worktree-Create` after the call; a `done` entry stays `done`; an absent slug creates no entry; a hub with no board config returns the same fabric-config error and writes nothing.
Each row asserts the closure's error is non-nil, which shows the claim precedes pair creation and survives the row's failure.
The test sets `BOARD_SKIP_GIT` and `BOARD_SKIP_PUSH`, process-global, so it does not call `t.Parallel` and says so in a comment.
Raise `internal/battencli`'s row in `test-budget.yaml` by one.

Docs: the `battencli` package comment in `cli.go` gains a paragraph on the claim write: when it is written, what it skips, that a failure warns, and that it stays on a failed Worktree-Create row.

**ImpactSummary:** batten's Worktree-Create row gains one board write before pair creation; a board failure only warns, so run behavior is otherwise unchanged.

**Commit:** 3: batten: claim the board entry before creating the pair
