---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/testkit/tmuxkit/tmuxkit.go:94
    summary: Publish verify failed in reedcli because afterRun found a sleep 300 left running after the sweep; the sweep does not itself guarantee that the swept servers' pane processes are gone
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/lyxcwd/lyxcwd.go:147
    summary: gitWorktreeRoot returns the lexical root dotgit.FindRoot walked up to, where the retired rev-parse returned git's symlink-resolved root, so a hub reached through a symlink now gets a different HubPath and reed socket key; no test covers a symlinked start
---

Review range: `git diff $(git merge-base main HEAD)..HEAD` (`product.parent` is `main`, and `_lyx/loom/rework/` does not exist).
The Publish failure section also names merge-in commit `7a87671cb`; its diff (`git diff 7a87671cb^1 7a87671cb`) is the parent branch's changes and was read for the cause of F1.
The focus directive for this round is empty and was followed as such.

### [BLOCKING] Publish verify fails: reedcli leaves a `sleep 300` after the post-run sweep

**Location:** `internal/testkit/tmuxkit/tmuxkit.go:94` (`afterRun`), `internal/testkit/tmuxkit/tmuxkit.go:186` (`sweep`); the failing package is `internal/reedcli`.
**Issue:** `.lyx/verify/publish-failure.log` ends the `reedcli` package with `PASS` and then `tmuxkit: process 3593964 left running by the package: ["sleep" "300"]`, `FAIL github.com/Knatte18/loomyard/internal/reedcli 49.553s`.
Every test passed; the package failed on the kit's end-of-run check, which is this branch's card 7.
The log shows no `straggling pane child` or `survived force-kill` warning, so reed's own reaping did not report a failure.

What the log and the code establish:

- A process counts as a leftover when its cwd or executable lies under the package's private directory, or its argv carries that directory (`referencesDir`).
  A bare `sleep 300` matches by cwd, so it is a pane process of a hub worktree under `TMPDIR`.
- The argv `["sleep" "300"]` is the command `smokeReapLaunchCmd` types into a pane shell, which the `TestSmoke*` scenarios in `internal/reedcli` use for most strands.
  The `reap-descendant` script's `sleep 300` has the same argv, but the `TestWatchdogReap` steps wait for the script's bash levels to die and never for the `sleep` itself.
- By pid arithmetic against the logged tmux server pids, the leaked process was started about 13:59:02.4, while `TestSmokeStateRecovery` was cycling servers on one hub socket (`recordedServerPID=3592600 liveServerPID=3594223`), concurrently with `TestWatchdogReap`.
- `sweep` runs `kill-server` and waits only until the socket refuses connections.
  It never confirms that the panes' process trees are gone, and so depends on the SIGHUP cascade landing within `leftoverGrace` (2 s) on a machine running every package at once.
  `afterRun` then kills the survivor, but still fails the package.

The merge-in did not change `tmuxkit` or the smoke tests, so the cause is not a merge conflict; it is load-dependent.
I could not reproduce it on this tree:
`lyx gate test --tags tmux ./internal/reedcli` passed once, `-run 'TestWatchdogReap|TestWatchdogDaemon' -count=2` passed three times, and `lyx gate test --tags tmux ./...` passed.
The exact test that started the process is therefore unproven.
The kit's message names only pid and argv, which is why: a leftover cannot be traced to a test from it.

**Fix:**

1. Make the sweep own the guarantee.
   Before `kill-server`, list each swept server's panes (`list-panes -a -F '#{pane_pid}'`) and take their descendant closure.
   After the kill, wait for those pids and SIGKILL any survivor, so a pane process the SIGHUP cascade is slow to reach, or cannot reach, is gone before the leftover scan starts.
2. Add the leftover's ppid and cwd to the `left running` line, so a recurrence names the test's hub.
3. Cover step 1 with a row in `TestAfterRun_LeftoverProcesses` or `TestSweep_KillsServersUnderItsDirectoryOnly`: a pane running a `trap '' HUP` sleep that survives `kill-server` alone.
4. Optionally, `DescendantClosureConfirmedExited` in `watchdogreap_integration_test.go` should record the `sleep` pid and wait for it, since it is the one process of that chain the step never asserts on.

### [MEDIUM] `lyxcwd` no longer canonicalizes the worktree root

**Location:** `internal/lyxcwd/lyxcwd.go:147` (`gitWorktreeRoot`), `internal/dotgit/dotgit.go:75` (`FindRoot`).
**Issue:** `gitWorktreeRoot` used `git rev-parse --show-toplevel`, which returns the symlink-resolved path (checked: from `link/sub` where `link -> real`, git prints `.../real`).
`FindRoot` returns the directory it found by walking up the lexical `start`, which is `lyxcwd.Getwd()` and so keeps a symlinked prefix.
`resolveCore` derives `HubPath` and `WorktreeName` from that root, and `reedengine.ServerName` hashes the hub path, so one hub reached through a symlinked and through its real path now gets two tmux socket keys and two reed servers.
The strict cwd gate hides the change, because `normalizePath` in `anchor.go` resolves symlinks on both sides before comparing.
`FindRoot`'s doc says it walks "as `git rev-parse --show-toplevel` does", and card 24's parity claim covers neither the symlink case nor a test row for it (`internal/dotgit/dotgit_test.go` and `internal/lyxcwd/lyxcwd_test.go` have none).
**Fix:** Resolve the root with `filepath.EvalSymlinks` in `gitWorktreeRoot` (or in `FindRoot`, keeping the `Geometry` as read), and add a symlinked-start row to `TestFindRoot` or the `lyxcwd` resolve table whose expectation is the real path.
