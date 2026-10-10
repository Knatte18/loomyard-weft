---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/testkit/tmuxkit/tmuxkit.go:94
    summary: afterRun's sweep leaves pane process trees to tmux's SIGHUP cascade, so a pane process that misses the hangup fails the package after the grace, and no test pins that case
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/lyxcwd/lyxcwd.go:148
    summary: gitWorktreeRoot resolves symlinks only in the root FindRoot found, not in the start it walked up from, so a symlinked start inside a repository resolves differently from the retired rev-parse, and no test or parity oracle covers it
---

### [MEDIUM] sweep does not guarantee the pane process trees are gone

**Location:** `internal/testkit/tmuxkit/tmuxkit.go:94` (`afterRun`), `internal/testkit/tmuxkit/tmuxkit.go:188` (`sweep`)

**Issue:**
`sweep` runs `kill-server` on each socket and `removeDeadSocket` waits until the server stops accepting connections.
Nothing in it snapshots or reaps the servers' pane processes.
The process trees are left to the SIGHUP cascade tmux starts when it closes the ptys, and `afterRun` then polls `leftovers(dir)` for up to `leftoverGrace` (two seconds).
A pane process that never receives the hangup is still there when the grace ends, so `afterRun` kills it and fails the package with exit 1.
That is what the Publish verify failure recorded: `tmuxkit: process 3593964 left running by the package: ["sleep" "300"]` on `internal/reedcli`.
The commit `15e310366` fixed that for reed's own `down` path by reaping the pane session's remaining members (`reapPaneChildren`, `sessionMemberPIDs`).
The kit's sweep, which every other tmux-tier package relies on, got no equivalent.
A test that drives a pane directly, or through `AddStrand`, and ends with the server alive until the sweep still has the same window: a command the pane shell forks after the hangup, or one not yet in the terminal's foreground, survives `kill-server`.
No test pins this.
`TestSweep_KillsServersUnderItsDirectoryOnly` starts a session with no pane command, and `TestAfterRun_LeftoverProcesses` passes a no-op sweep, so a pane process that survives `kill-server` alone, for example a `trap '' HUP` sleep, is never exercised.
I ran `lyx gate test --tags integration,tmux ./internal/reedcli` at HEAD and it passes, so the recorded failure is fixed for reed's path; the kit-level gap remains.

**Fix:**
Make `sweep` reap the process trees itself.
Before each `kill-server`, read the server's pane pids (`list-panes -a -F '#{pane_pid}'`), and after it kill every remaining process in a session those pids led, as `reapPaneChildren` does.
Add a case to `tmuxkit_integration_test.go` that starts a pane running `trap '' HUP; sleep 300` on a kit server, runs `sweep`, and asserts the sleep is gone with no leftover finding.

### [MEDIUM] gitWorktreeRoot resolves the root but not the start, so a symlinked start diverges from rev-parse

**Location:** `internal/lyxcwd/lyxcwd.go:148` (`gitWorktreeRoot`), `internal/dotgit/dotgit.go:70` (`FindRoot`), `internal/lyxcwd/lyxcwd_test.go` (`TestResolve_AnchorScenario`, step "symlinked start")

**Issue:**
`git rev-parse --show-toplevel` resolves the working directory first, then finds the toplevel, so a start reached through a symlink gives the real root.
The replacement calls `dotgit.FindRoot(cwd)`, which climbs lexically with `filepath.Dir` from the path as given, and only afterwards runs `filepath.EvalSymlinks` on the root it found.
This agrees with git only when the symlink is the root itself, the one case the new test row covers (`link -> root`).
When the start is a symlink to a subdirectory of a repository, or has a symlinked ancestor below the root, the lexical parents are not the real parents.
For `/a/link -> /b/repo/sub` the walk visits `/a/link`, `/a` and `/`, never reaches `/b/repo`, and returns `ErrNotAGitRepo` where git resolved the repository, or it finds an unrelated repository that encloses `/a`.
`os.Getwd` returns `$PWD` when it names the current directory, so a shell standing in a symlinked path produces exactly this start.
A wrong or missing root changes `HubPath` and the reed socket key, the consequence `bc0864a4b` set out to prevent.
`dotgit.FindRoot` has no test starting from a symlink, and the oracle comparison for the toplevel read was dropped with `Repo.Toplevel` (commit `8cdeaffc1`), so nothing compares `FindRoot` to `rev-parse --show-toplevel`.
The same-path claim in the `gitWorktreeRoot` comment, "A worktree reached through a symlinked path thus gets the same hub path", is therefore only true for the root-symlink shape.

**Fix:**
In `gitWorktreeRoot`, resolve `cwd` with `filepath.EvalSymlinks` before `dotgit.FindRoot`, and return the root `FindRoot` gives from the resolved start, so the root needs no second resolution.
Extend the "symlinked start" step of `TestResolve_AnchorScenario` with a row whose link points at a subdirectory of the checkout and expects the same `Location` as `Resolve(root)`.
Add a `dotgit` test row, or a `lyxcwd` row against `git rev-parse --show-toplevel` through the existing oracle, so the symlinked start is compared to git directly.
