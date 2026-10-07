# Card 8 — notifier-wiring

**Create:**
- `internal/orchcli/notify.go`
- `internal/orchcli/notify_integration_test.go`

**Edit:**
- `internal/loomcli/wiring.go`
- `internal/loomcli/landingdeps.go`
- `internal/webstercli/wiring.go`
- `internal/burlercli/wiring.go`
- `internal/orchcli/doc.go`

**Uses:**
- `internal/shuttleengine#Runner.SetNotifier`
- `internal/orchengine/notice.go`
- `internal/battencli/wire.go`

**Intent:**
D5, the wiring half: the hub-mode runner each of `loomcli`, `webstercli` and `burlercli` builds notifies the prime's orch through its notice queue.

`orchcli/notify.go` adds one exported function that takes a worktree's `*lyxcwd.Location` and a line, resolves the hub's prime exactly as `loomcli/landingdeps.go`'s `Notify` closure does today (`fabricengine.PrimeName`, `fabricengine.WorktreePath`, `lyxcwd.ResolveWorktree`), and queues the line with `orchengine.QueueNotice(PrimePaths(prime), line, time.Now())`.
`landingdeps.go`'s `Notify` closure calls it instead of repeating the resolution.
A notice the orch has no strand to receive is logged by `QueueNotice` and dropped, as for batten's notices.

The hub-mode wiring of `loomcli` (`wiring.go`, the runner shared by burler rounds, webster's Master, friction reflection, landing merge-resolve and the driver starter), `webstercli` (its hub runner, which carries `lyx webster recover-batch`'s recovery strand) and `burlercli` (its hub runner, the operator's `lyx burler`) calls `SetNotifier` with a closure over that function and the command's location.
`PATTERN-orch-pane-single-writer` and `PATTERN-told-geometry` hold: the engine gets a function, never a queue path.
The runners `shuttlecli` and `orchcli` build, and every detached (standalone) runner, wire no notifier.
Add one sentence to `orchcli/doc.go`'s notice paragraph naming the new function as the way a task-worktree module reaches the prime's queue.

Test: `notify_integration_test.go` (`integration` tag, `internal/hubforge` fixture through `fabriccli.CloneAndWire`, per `PATTERN-hubforge-fixtures`) calls the function from a task worktree's location with an orch strand recorded in the prime's orch state, and asserts exactly one notice file holding the line in the prime's notices directory and none in the task worktree's; with no strand recorded it queues nothing and returns no error.
Check that `internal/orchcli`'s `TestMain` runs through `tmuxkit.Main` (`PATTERN-test-isolation`).

**ImpactSummary:** Hold notices from loom, webster and burler runs reach the prime's orch; landing's notify path reuses the same resolution.
