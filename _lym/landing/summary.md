# Guard every pinned lym command against launching a test binary

A command that lym pins into tmux or hands to a detached process used to name `os.Executable()`.
Under `go test` that path is the test binary.
A test binary given a lym verb such as `slack bar` runs its whole suite, and each tmux session that suite starts runs the same command again.
On 2026-11-08 this fork bomb froze the development machine twice, with hundreds of `reedengine.test`, `tmux` and `bash` processes.
An earlier fix covered only reed's Slack lines.
This change guards the whole class.

## What changes

- `internal/selfexec` gains `LymBinary` and `ResolveLymBinary`, the one place that resolves "this lym binary".
  Under `go test` they return `ErrTestBinary` for the running test binary, and for any path when the test binary's own path does not resolve.
  `ResolveLymBinary` takes an injectable path lookup, so a test's stand-in path still passes.
- Every pin or spawn that ran a pinned or detached lym command now resolves through them.
  The sites are in `reedengine` (pane prelude, window, switch, follow, button and Slack lines, the spawn watchdog), `reedcli`, `loomcli`,
  `battencli`, `adoptcli`, `boardengine`, `creelcli`, `driftcli`, `fabriccli`, `fabricengine`, `fronttask`, `gatecli`, `ideengine` and `orchcli`.
  `reedengine`'s private `slackLineExecutable` is removed in favour of the shared helper.
- `tmuxkit.Main` refuses a test binary started with a positional first argument.
  It writes the refusal to stderr, removes the run directory, and returns 2.
  This replaces the guards in `gitkit` (`refuseCLIReexec`) and `reedengine`'s `TestMain`, so every package that calls `tmuxkit.Main` gets it.
  Only the first argument is read, since a later word may be a flag's value, as in `-test.run Foo`.
- `lym gate test` caps the processes a tier run may hold.
  Every process of a run carries the run's enclosing-gate environment marker, and `gatecli` counts them through the new `proc.PIDsWithEnv`, which reads `/proc` on Linux.
  Past `runProcessCap` (1024) it kills them all with SIGKILL and fails the tier with a line naming `selfexec.LymBinary` as the way forward.
  A capped run is never rerun as a resource shortage.
  Other platforms log that no cap applies and run uncapped.
- `PATTERN.md`, the `pattern/` documents and `docs/overview.md` describe the helper, the `TestMain` guard and the cap.

## Design decisions

- The refusal lives in `selfexec` and not in each caller, so a new pin cannot forget it.
  The injectable form keeps the existing path-lookup parameters of the compose functions, with tests supplying a stand-in path.
- The guard moved into `tmuxkit.Main` and out of `gitkit.HermeticGitEnv`, because `Main` is the call every test package already makes last.
  `cmd/lym`'s hermetic guard still scans for the `HermeticGitEnv` token, which is unchanged.
- The process cap keys on an environment marker and not on the process tree, because tmux server panes leave the run's tree.
  The cost is a scan of `/proc` once per sample period.

## To check by hand

- `os.Executable` is still called directly in `hubgeom`, `fabricengine/stencilcommit.go`, `statuspane`, `loomcli/drivewiring.go`, `tendercli`,
  `reedcli/watchdog.go` and `selfexec/selfexec.go`.
  I did not verify that none of these pins a command a tmux job or detached process runs; confirm each is a read-only use, as `selfexec/doc.go` says.
- The process cap's kill path ran only against fakes in unit tests.
  Running `lym gate test` against a deliberately looping test on a Linux machine would show it fires and fails the tier before the machine strains.
  Do this only on a machine you can afford to stress.
