# Card 13 — reed-window-verb

**Create:**
- `plan:internal/reedcli#reedCLI.windowCmd` -> `func (c *reedCLI) windowCmd() *cobra.Command`
- `plan:internal/reedcli#TestWindowCmd_OpensATerminalAttachedToTheToldWorktree` -> `func TestWindowCmd_OpensATerminalAttachedToTheToldWorktree(t *testing.T)`

**Edit:**
- `internal/reedcli#reedCLI`
- `internal/reedcli#Command`
- `internal/reedcli/doc.go`

**Uses:**
- `internal/termwindow#Resolve`
- `internal/termwindow#Launcher.OpenAttach`

**Intent:** Give Open-Window's action a `lyx` verb: `lyx reed window <worktree-dir> [--title <title>]` opens one desktop terminal window attached to that worktree's reed session through `internal/termwindow` (D4).

`windowCmd` is an operator-audience verb taking exactly one positional, the worktree directory the window's shell runs `lyx reed attach` in, and `--title`, defaulting to the directory's base name.
It resolves the running `lyx` path with `os.Executable` and calls `c.openTerminal(dir, title, lyxPath)`, which defaults to `termwindow.Resolve()` then `Launcher.OpenAttach`, so it returns once the launcher process has started and the window outlives the caller.
`reedCLI` gains that `openTerminal` seam, nil meaning the real launcher.
A directory that does not exist is refused before any launcher runs, naming the path; a machine with no launcher returns the `termwindow.ErrNoLauncher` error on the envelope, naming why.
Its envelope on success is `{"dir": <dir>, "title": <title>}`.
`Command` registers it beside `attach`, and its pre-run's skip list, which today holds `reed`, `watchdog`, `switch` and `follow`, gains `window`: the verb needs no location, config or engine, and resolving one would refuse a worktree directory named from outside the hub's worktrees; `reedcli/doc.go` lists it.
The verb opens a window on any directory the caller names; what bounds it is that the window runs only `lyx reed attach` there, which attaches to that worktree's own reed session and nothing else.

`TestWindowCmd_OpensATerminalAttachedToTheToldWorktree`, Tier 1 on a `reedCLI` with a recording `openTerminal`, runs the command's `RunE` through its cobra tree: a temp directory with `--title t` records one call with that directory, `t` and a non-empty lyx path; the same directory with no `--title` records its base name as the title; a missing directory exits non-zero with no call; and an `openTerminal` returning an error exits non-zero carrying it.

**ImpactSummary:** One new reed verb and one nil-default seam field; no existing verb changes.

**Commit:** 13: lyx reed window
