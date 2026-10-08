# Card 4 — Every git remote command runs under a deadline and a low-speed limit

**Create:**
- `plan:internal/gitexec#remoteDeadline` -> `var remoteDeadline = 10 * time.Minute`
- `plan:internal/gitexec#remoteWaitDelay` -> `const remoteWaitDelay = 10 * time.Second`
- `plan:internal/gitexec#lowSpeedEnv` -> `var lowSpeedEnv = []string{"GIT_HTTP_LOW_SPEED_LIMIT=1000", "GIT_HTTP_LOW_SPEED_TIME=60"}`
- `plan:internal/gitexec#SetRemoteDeadlineForTest` -> `func SetRemoteDeadlineForTest(t *testing.T, d time.Duration)`
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

`gitexec` cannot import `internal/logger` (`logger` reaches `gitexec` through `lyxcwd`), and `cmd/lyx/spawnobservability_test.go` already records that exemption; the kill's `report` callback does nothing, and the timeout reaches the logs through the `*GitError` every caller logs.
The new `doc.go` carries the package doc: the `Run`/`RunGit` split, the remote bounds and their limits, the Windows limit that a timed-out git's `git-remote-https` child may outlive the kill, and the logging exemption's reason.

Tests:

- `transport_test.go`: rows for a `pull` failing with a transport phrase, a remote command whose stderr carries `Operation too slow`, and a remote `*GitError` with `Timeout` set, each true.
- `errorrender_test.go`: a row for a timed-out `GitError`'s rendering.
- `gitexec_test.go`: one new integration test, not parallel because it sets `remoteDeadline` (process-global state, named in a comment), drives git's `ext::` transport (`-c protocol.ext.allow=always`) as a remote that accepts and never answers, with no network.
  Rows: an `ls-remote` whose `ext::` command records its pid and sleeps fails within a short injected deadline as a `*GitError` with `Timeout` set that `IsTransportFailure` reports true for, and on Unix the recorded pid is no longer alive (`proc.IsAlive`); an `ls-remote` whose `ext::` command prints its environment to stderr shows both low-speed variables; a local command (an alias running a shell that sleeps past the deadline and prints its environment) succeeds and shows neither.
- `cmd/lyx/testdata/test-budget.yaml`: raise `internal/gitexec`'s row by one.

**ImpactSummary:** Every remote git call under `internal/` gains a deadline, a process-group kill and the low-speed environment, and a timed-out or too-slow one now classifies as a transport failure; local calls and every call site are unchanged.

**Commit:** 4: gitexec: bound every git remote command
