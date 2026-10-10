# Card 4 — Every git remote command runs under a deadline and a low-speed limit

**Create:**
- `plan:internal/gitexec#remoteDeadline` -> `var remoteDeadline = 10 * time.Minute`
- `plan:internal/gitexec#remoteWaitDelay` -> `const remoteWaitDelay = 10 * time.Second`
- `plan:internal/gitexec#lowSpeedEnv` -> `var lowSpeedEnv = []string{"GIT_HTTP_LOW_SPEED_LIMIT=1000", "GIT_HTTP_LOW_SPEED_TIME=60"}`
- `plan:internal/gitexec#SetRemoteDeadlineForTest` -> `func SetRemoteDeadlineForTest(t *testing.T, d time.Duration)`
- `plan:internal/gitexec#SetKillReporter` -> `func SetKillReporter(report func(args []string, pid int, groupErr error))`
- `internal/gitexec/doc.go`

**Edit:**
- `internal/gitexec#GitError`
- `internal/gitexec#GitError.Error`
- `internal/gitexec#runCore`
- `internal/gitexec#RunGit`
- `internal/gitexec#Run`
- `internal/gitexec#remoteSubcommands`
- `internal/gitexec#transportPhrases`
- `internal/gitexec#IsTransportFailure`
- `internal/gitexec/gitexec_test.go`
- `internal/gitexec/transport_test.go`
- `internal/gitexec/errorrender_test.go`
- `cmd/lyx#main`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/gitexec#subcommand`
- `plan:internal/proc#ConfigureGroupKill`

**Intent:** Decision D3 (#458): a git network call can hang forever on a dead connection after a laptop sleep.
All git under `internal/` runs through `runCore`, so one change there bounds every remote call lyx makes, with no per-caller wiring and no change at any `Run`/`RunGit` call site (`PATTERN-gitexec-checked-call`).

`remoteSubcommands` gains `pull`, so it holds `push`, `fetch`, `pull`, `clone` and `ls-remote`; it is now both the set `runCore` bounds and the set `IsTransportFailure` classifies.
When `subcommand(args)` is in that set, `runCore`:

- runs git under `exec.CommandContext` with a `remoteDeadline` timeout, with `proc.ConfigureGroupKill` so expiry kills git's process group on Unix and git alone on Windows, and with `cmd.WaitDelay = remoteWaitDelay` bounding the wait on pipes an orphaned helper such as `git-remote-https` still holds;
- appends `lowSpeedEnv` to `os.Environ()` as `cmd.Env`, so a stalled HTTPS transfer fails within about a minute.

A local subcommand runs exactly as today: no deadline, no process group, inherited environment.
On expiry `runCore` returns exit code -1 and a non-nil `*GitError` carrying the args, dir, captured stderr and the deadline, so `RunGit` hands it back as its error and `Run` returns it as is.
`GitError` gains a `Timeout time.Duration` field, zero unless the deadline killed git, and `GitError.Error` renders `git <args>: timed out after <d>`, followed by the trimmed stderr when non-empty, in place of the exit code.
`Run`'s and `RunGit`'s docs, and the file comment, state that a timed-out remote command is a `*GitError` though git did not exit on its own; an exec-level failure stays unwrapped.

`IsTransportFailure` reports true for a remote subcommand's `*GitError` whose `Timeout` is non-zero, and `transportPhrases` gains `operation too slow`, curl's text when the low-speed limit fires.
So every caller's existing transport path takes over: a best-effort push logs and continues, and a shed boundary's transient mark gets its one re-step (`PATTERN-transient-stop`; no new mark).
Bound, which the doc states: a legitimate remote command that runs longer than 10 minutes, or moves under 1000 B/s for 60 s, now fails as a transport failure, and the worst case is one deadline per remote command a step attempt runs, times two attempts.
The deadline and limit are not operator configuration; `remoteDeadline` is a variable only so a test can shorten it through `SetRemoteDeadlineForTest`, which lives in a new `export_test.go` in package `gitexec` and restores the value in `t.Cleanup`.

The decision record's `PATTERN-spawn-observability` constraint logs a timed-out git kill through `internal/logger`, but `gitexec` cannot import `logger` (`logger` reaches `gitexec` through `lyxcwd`), the exemption `cmd/lyx/spawnobservability_test.go` already records.
So the kill's outcome goes to a package-level kill reporter: `runCore`'s `report` callback for `proc.ConfigureGroupKill` hands the args, the pid and the group kill's error to it.
`SetKillReporter` installs it; the default, which a nil reporter restores, drops the outcome.
`cmd/lyx`'s `main` installs a reporter once, before the root command runs, that logs through `internal/logger`: an info line naming the args and pid when the group kill succeeded, and a warning when `groupErr` is non-nil (the group kill failed and git was killed alone, or `proc.ErrNoProcessGroup` on Windows), since a descendant such as `git-remote-https` may then outlive git.
Bound: the reporter receives only a kill's outcome, after the kill; it cannot stop, delay or retry the kill or change `runCore`'s result, and `SetKillReporter` is called only from `main` and from gitexec's own non-parallel test, never from a module or a test run in parallel.
The new `doc.go` carries the package doc: the `Run`/`RunGit` split, the remote bounds and their limits, the Windows limit that a timed-out git's `git-remote-https` child may outlive the kill, and why the kill's outcome reaches the logs through the kill reporter rather than an import of `logger`.

Tests:

- `transport_test.go`: `TestIsTransportFailure`'s existing `pull with phrase` row flips its expectation to true; curl's `fatal: unable to access 'x': Operation too slow` joins the `phrases` list the test iterates over push and fetch; and a new `cases` row, a remote `*GitError` with `Timeout` set and no transport phrase in its stderr, is true.
- `errorrender_test.go`: a row for a timed-out `GitError`'s rendering.
- `gitexec_test.go`: one new integration test, not parallel because it sets `remoteDeadline` and the kill reporter (process-global state, named in a comment), drives git's `ext::` transport (`-c protocol.ext.allow=always`) as a remote that accepts and never answers, with no network.
  Rows: an `ls-remote` whose `ext::` command records its pid and sleeps fails within a short injected deadline as a `*GitError` with `Timeout` set that `IsTransportFailure` reports true for, a recording reporter the test installs through `SetKillReporter` (restoring the default in `t.Cleanup`) receives the kill's args and a nil group error, and on Unix the recorded pid stops being alive (`proc.IsAlive`), polled to a bounded deadline as `verifyrun`'s `TestRun_ExpiryKillsBackgroundedDescendant` does, since the killed child is briefly a zombie until it is reaped; an `ls-remote` whose `ext::` command prints its environment to stderr shows both low-speed variables; a local command (an alias running a shell that sleeps past the deadline and prints its environment) succeeds and shows neither.
- `cmd/lyx/testdata/test-budget.yaml`: raise `internal/gitexec`'s row by one.

**ImpactSummary:** Every remote git call under `internal/` gains a deadline, a process-group kill and the low-speed environment, and a timed-out or too-slow one now classifies as a transport failure; a kill's outcome is logged through the reporter `main` installs; local calls and every call site are unchanged.

**Commit:** 4: gitexec: bound every git remote command
