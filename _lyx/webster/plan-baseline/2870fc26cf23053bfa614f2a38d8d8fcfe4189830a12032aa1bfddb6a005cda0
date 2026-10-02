# Card 14 — reed-list-verb

**Create:**
- `internal/reedengine/directory.go`
- `internal/reedengine/directory_test.go`
- `internal/reedcli/list.go`
- `internal/reedcli/list_integration_test.go`

**Edit:**
- `internal/reedcli/cli.go`
- `internal/reedcli/cli_test.go`
- `cmd/lyx/helptree_test.go`

**Uses:**
- `internal/hubgeom/hubgeom.go`
- `internal/fabricengine/worktreelist.go`
- `internal/reedengine/namerepair.go`

**Intent:**
Add `lyx reed list`, the hub-wide name directory and the diagnostic for the two mirrors.

`(e *Engine) Directory() ([]DirectoryRow, error)` reads this worktree's state under the op lock without requiring a session: with no session up every row is dormant with an empty title;
otherwise it lists panes once and fills each row.
`DirectoryRow` carries `Name`, `GUID`, `Worktree` (the stamped `Strand.Worktree`), `PaneID`, `Title` (the pane's current title, empty when not live), `Live`, and `Drift`, true when the strand is live and its title differs from its name — the same comparison `planTitleRepairs` makes, called rather than restated.
It is read-only: no repair, no persist.

`reedcli/list.go`: `list` enumerates the hub's warp worktrees with `fabricengine.List(location.AnchorPath())`, resolves each with `lyxcwd.ResolveWorktree`, builds its engine from `reedengine.LoadConfig` and `hubgeom.ReedGeometry`, and concatenates every worktree's `Directory()` rows, sorted by name.
A worktree that fails to resolve or read is skipped with a named `logger.Warn`, so one broken worktree never hides the rest.
It emits the standard JSON envelope, `{"ok":true,"strands":[{"name":…,"guid":…,"worktree":…,"pane_id":…,"title":…,"live":…,"drift":…}]}`, carries a `Short`, takes no positional arguments, and checks `clihelp.ShouldAbort` first.
`cli.go` registers it; both help-tree listings gain `list`.

`directory_test.go` on the fake tmux: a live matching strand, a live drifted one, a dead-pane one and an unbound one map to the right rows, and a cold session reports every row dormant without a tmux session error.
`list_integration_test.go` (`integration` tag) on a `hubforge` hub with a task worktree: strands added in the prime and in the task worktree all appear in one `list` from the prime, each with its own worktree, full name, pane id, title and liveness; retitling one pane by hand flips only that row's `drift`.

**ImpactSummary:** Adds one read-only verb and one read-only engine method; existing verbs and the watchdog are untouched.

**Commit:** 14: reed list: the hub-wide name directory
