# Card 8 — batten-notices

**Edit:**
- `internal/battenshed/deps.go`
- `internal/battenshed/innerrun.go`
- `internal/battenshed/doc.go`
- `internal/shedrecipe/entries_batten.go`
- `internal/shedrecipe/entries_batten_test.go`
- `contracts/recipes/batten-recipe.yaml`

**Create:**
- `internal/battenshed/notice.go`
- `internal/battenshed/notice_test.go`

**Intent:**
Make the Run-Shed row write run notices (D12), through a seam so `battenshed` keeps its import allowlist (shedengine and logger only).

`InnerRunDeps` gains `Notify func(ctx context.Context, line string) error`, `NoticeQuiet time.Duration` and `AttachDir func() (string, error)` (the task worktree directory the attach command `cd`s into, resolved on Call like the other seams).
A nil `Notify` resolves to a no-op in `NewInnerRun`, the same way a nil `Sleep` does, so existing tests and wirings stay valid.
A `Notify` error is only warned about and never changes the row's outcome.

`notice.go` holds the pure rendering and the episode bookkeeping:
- A notice is one line: the literal prefix `[batten notice]`, the slug, the condition, the child state, the child's error if any, and `cd <dir> && lyx reed attach`.
  The error has newlines and control characters replaced by spaces and is cut to 200 characters, so a notice never begins with `/` and has no second line.
- Conditions: the child's state changed away from running (awaiting, blocked, paused, failed, done); the driver strand found dead while the child is running; the child's status file unchanged for `NoticeQuiet` while the child is running and the driver strand is alive.
- One notice per episode.
  An episode ends when the child returns to running or the status file changes (its modification time), so each condition notifies once per episode.
  Episode state is a marker file under the row's scratch directory, beside the existing once-markers, keyed by condition, child state and status-file modification time, so a batten restart does not notify an episode already notified.

`innerrun.go` calls the notice step after each status read: the state-change condition in every non-running arm, and the dead-driver and quiet conditions in the running arm only, which then calls `DriverAlive` (already wired) and stats the status file.
Bound: notices are informational; the row's outcomes, bounce budget, halts and waits are unchanged, and the orch acts on a notice only through `lyx batten status <slug>`.

Config: `innerRunEntry` reads `notice_quiet_min` like `poll_interval_s` (non-negative, zero or absent resolves to a default of 45), sets `NoticeQuiet` on the deps it passes to `NewInnerRun`, and accepts the key in `configRejectUnknown`; `batten-recipe.yaml`'s Run-Shed row sets `notice_quiet_min: 45`.

Tests in `notice_test.go`: rendering (prefix, one line, control characters replaced, 200-character cut, attach command); each condition writes one notice per episode; the quiet condition fires only for a running child with a live driver; a status change or a return to running starts a new episode; a fresh producer over the same scratch directory (a restart) does not re-notify.
`entries_batten_test.go`: `notice_quiet_min` defaults to 45, a negative value errors naming the key.

`internal/battenshed/doc.go`: rewrite the paragraph saying a dead, parked or stopped driver is not detected here, to say the row now notices a dead driver and a quiet status file and notifies, while control flow is unchanged.

**ImpactSummary:** Adds three InnerRunDeps fields and a notice step to the Run-Shed row plus one recipe config key; the row's outcomes and budgets are unchanged and a nil Notify keeps existing wirings valid.
