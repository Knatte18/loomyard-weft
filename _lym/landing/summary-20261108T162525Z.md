# Keep test binaries out of lym's pins, and cap a gate run's processes

A command lym builds from its own binary path and hands to tmux or another process runs the test binary when the builder runs under `go test`.
That test binary ignores the lym verb, runs its whole suite, and each tmux session the suite starts can pin the same command again.
On 2026-11-08 that loop froze the machine twice: hundreds of test binaries, load above 800, memory and swap full.
`3aaa98cc` closed the one route that did it, reed's Slack lines.
This change closes the whole class with three independent layers.

## What changes

- **One lookup of "this lym binary".**
  `selfexec.LymBinary` resolves the running binary for a command another process runs later, and returns `selfexec.ErrTestBinary` under `go test`.
  `selfexec.ResolveLymBinary` applies the same refusal to an injectable lookup, so a test's stand-in path still passes while `os.Executable` is refused.
- **Reed's pins all go through it.**
  The strand-pane prelude, the detached window command, the session-switch and follow bindings, the button click and its prompt re-entry, both Slack lines and the watchdog spawn.
  `Engine.executablePath` and the button deps now default to `selfexec.LymBinary`.
  `slackLineExecutable` is gone; its check now covers every pin.
- **The other pins and spawns go through it too:**
  loom's status strand, Go driver spawn, drift capture, raddle's `lym` child and loop spec;
  orch's runs strand and watcher;
  batten's loop spec, `loom start` child and detached pass;
  adopt's check and resume child;
  fabric's detached push and teardown hold;
  fabric anchor's `beam start` child; drift's capture child; the front task's land action.
- **`tmuxkit.Main` refuses a lym verb.**
  Every package's `TestMain` calls it, and it now returns 2 before any test runs when the binary's first argument is not a flag.
  A pin that slips past the lookup, in any package, makes the test binary exit at once instead of running its suite.
  Only the first argument is read, since a later word may be a flag's value.
  This replaces gitkit's copy of the guard in `HermeticGitEnv` and reedengine's own `TestMain` check.
  The end-to-end test moves to tmuxkit.
- **`lym gate test` caps a tier run's processes.**
  Once per sample period the verb lists every process whose environment carries the run's `LYM_GATE_TEST_PID` entry (`proc.PIDsWithEnv`).
  A tmux server a test starts keeps that entry in its panes, so processes that left the command's tree still count.
  Past `runProcessCap` (1024) it SIGKILLs every listed process, and does so again each period until the command ends.
  The tier fails with a line naming the cap and the way forward, and is never rerun as a resource shortage.
  Off Linux the cap logs that none applies.
- **Docs:** `PATTERN-child-processes` and its `PATTERN.md` entry name `selfexec.LymBinary`;
  `PATTERN-test-isolation` states the `tmuxkit.Main` refusal;
  `PATTERN-leaf-packages`, the `selfexec`, `reedengine`, `proc` and `gatecli` package docs and the overview row are updated.

## Design decisions

- **The helper lives in `selfexec`, not a new package.**
  `selfexec` already owns the running binary's path for re-exec and is a leaf every spawning package can import.
  The name `lymbin` is taken by the testkit that builds `lym`.
- **One guard in `tmuxkit.Main` rather than one per package.**
  The board entry asks for a `TestMain` guard and a proving test in every package whose tests reach a pin.
  `PATTERN-test-isolation`'s enforcement (`cmd/lym/tmuxisolation_test.go`, empty allowlist) already makes every test package call `tmuxkit.Main`.
  So the guard there covers every package by construction, and one test in tmuxkit proves it,
  without one subprocess test per package.
  Every `TestMain` that serves helper roles of its own already dispatches them before calling `tmuxkit.Main`.
- **Sites left on `os.Executable`:**
  - Reads of the binary itself: `fabricengine.BinaryLabel`, `statuspane.CurrentBuild`, adopt's running-build read, tender's stale-process match and the VS Code config path.
  - `hubgeom.DobbyGeometry` and `hubgeom.SlackGeometry`: geometry is built under test in many places, and their spawn sites in `dobbyengine` and `slackengine` already refuse under `go test`.
  - The resolve-once paths in loom's `goDriverDepsFor` and batten's `wire`: they also feed the on-disk build identity, and a refusal there would fail every wiring test.
    Their spawns run only in a live driver, and the `tmuxkit.Main` guard backstops them.
- **The cap is a constant, not a `gate.yaml` key.**
  Normal tier runs stay far below 1024 marked processes, and the cap exists to stop a runaway, not to tune.
- **Counting by the environment entry, not the process tree.**
  The fork bomb ran through a tmux server, which daemonizes out of the tier command's tree, so a tree walk would not see it.

## To check by hand

- Run `lym gate test --tier 2` for `internal/reedengine`, `internal/loomcli` and `internal/orchcli`.
  Some tests may have relied on the test binary's path appearing in a pane prelude or binding when `Engine.New`'s default lookup was used.
- Watch a full gate run's peak count of marked processes (`grep -l LYM_GATE_TEST_PID=<pid> /proc/*/environ | wc -l`) to confirm the cap's headroom on this machine.
