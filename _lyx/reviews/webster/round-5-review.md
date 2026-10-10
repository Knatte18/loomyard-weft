---
verdict: APPROVED
---

# Round 5 review

Review range: no `_lyx/loom/rework/` round exists, so the range is `git diff $(git merge-base main HEAD)..HEAD`, with `product.parent` read from `_lyx/shed/test-suite-weight/status.json`.
Prior-round files were not read.
The focus directive's two questions were settled against the code at HEAD (`acfdae95b`); neither raises a finding.

## Publish failure

The log at `.lyx/verify/publish-failure.log` names one failing package, `internal/reedcli`, with `tmuxkit: process 3593964 left running by the package: ["sleep" "300"]`.
That message has the pre-fix format (no cwd), so the log predates the later fixes.
`.lyx/verify/verified-tree.yaml` records the full verify command, including `go test -tags tmux ./...`, passing on commit `acfdae95b` (HEAD), and `verify.log` shows `reedcli` passing in all three tiers.
I re-ran `lyx gate test --tags tmux ./internal/testkit/tmuxkit` and `lyx gate test ./internal/lyxcwd ./internal/dotgit`; all three packages pass.
The failure's cause (a pane's SIGHUP-immune or late-forked `sleep 300` outliving `kill-server`) is closed by `killServer` in `internal/testkit/tmuxkit/tmuxkit.go:200` for the kit and by `reapPaneChildren` in `internal/reedengine/lifecycle.go` for reed's own destroy path.
No publish-failure finding stands.

## Focus 1 — `sweep` and `afterRun` in `internal/testkit/tmuxkit/tmuxkit.go`

Verdict: not a defect.

- `sweep` runs `killServer` for every socket under the kit directory.
  `killServer` snapshots the pane pids with `list-panes -a`, sends `kill-server`, waits for the socket to refuse connections, then `killSessionMembers` force-kills every process whose `/proc/<pid>/stat` session id is a snapshotted pane pid.
  tmux makes each pane's process a session leader, so a pane child that ignores SIGHUP, or that never reached the terminal's foreground, stays in that session and is killed.
- `KillOnCleanup` (and so `Socket`) registers the same `killServer`, so a per-test server is reaped by the same path.
- `afterRun` is the backstop: after `sweep` it polls `leftovers(dir)` for the grace period, kills each survivor whose cwd, executable or argv references the kit directory, and fails the package for each that is not a reed watchdog.
  A process that escapes the session and references the directory therefore still fails the run loudly; the kit's private `TMPDIR` puts fixture cwds under that directory.
- `TestSweep_KillsServersAndPaneTreesUnderItsDirectoryOnly` (`tmuxkit_integration_test.go:100`) starts a pane running `trap '' HUP; sleep <unique> & wait`, so both the shell and the `sleep` ignore the hangup.
  It runs `sweep` and asserts the `sleep` process is gone, the server and socket file under the swept directory are gone, and a second server and its socket outside the directory are untouched.
  It does not call `afterRun`, but the payload's absence is the stronger assertion than "no leftover finding", and `TestAfterRun_LeftoverProcesses` pins the leftover finding and the kill separately.

Residual reach, accepted: the kill is one pass over a `/proc` snapshot, and a process that calls `setsid` leaves the session.
Both need a pane command that keeps forking, or detaches on purpose, while ignoring the hangup and without referencing the kit directory.
No test or production pane in the repo does that, and the `afterRun` scan covers the directory-referencing cases, so I do not raise it.

## Focus 2 — `gitWorktreeRoot` and `dotgit.FindRoot`

Verdict: not a defect.

- `gitWorktreeRoot` (`internal/lyxcwd/lyxcwd.go:148`) calls `filepath.EvalSymlinks(cwd)` and walks up from the resolved path with `dotgit.FindRoot`.
  `FindRoot` itself walks lexically and its doc says so; its only production caller resolves first.
  Git calls `getcwd()` and so walks the real path; I ran `git rev-parse --show-toplevel` from a directory with a symlinked ancestor below the root, from a symlink to a subdirectory, and from a symlink to the root, and each printed the real root, which is what the resolve-then-walk returns.
- The `symlinked start` subtest in `TestResolve_AnchorScenario` (`internal/lyxcwd/lyxcwd_test.go`) links the root and a nested subdirectory with `fslink.CreateDirLink` and asserts `Resolve` and `ResolveWorktree` return the real root's `Location`.
  The link to a subdirectory is the case a lexical walk gets wrong, because no repository lies among its lexical parents.
  The test compares to the fixture's real `Location`, not to a `git rev-parse` subprocess; for a fixture created by `gitkit`, the real root is git's answer, and a subprocess comparison would add a test for no new coverage (`PATTERN-test-economy`).
- `checkCwdAnchorGate` compares through `samePath`, which `EvalSymlinks`-normalizes both sides, so the unresolved `cwd` passed to `buildLocation` does not make the gate disagree with the resolved root.

## Other checks

- `PATTERN-cwd-resolution`, `PATTERN-leaf-packages`: the production importers of `internal/dotgit` are `internal/gitrepo` and `internal/lyxcwd`, and the diff adds no `os.Getwd` or `--show-toplevel` call in non-test code, only comments naming git's behavior.
- Comment conventions: new doc comments in `tmuxkit.go`, `proc.go`, `dotgit.go` and `lyxcwd.go` state purpose and contract without history; line breaks are not judged here (`PATTERN-comment-line-breaks`).
- `tmuxkit.procSession` and `reedengine.parseStat` both parse the session id out of `/proc/<pid>/stat`.
  The duplication is deliberate: `tmuxkit` cannot import `reedengine`, whose tests import the kit, and `serverEnviron` documents the same constraint for its copy.
