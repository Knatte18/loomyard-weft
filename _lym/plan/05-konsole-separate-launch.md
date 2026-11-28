# Card 5 — konsole-separate-launch

**Create:**
- `internal/termwindow#Launcher.IsKonsole`
- `internal/termwindow#Launcher.Open`
- `internal/termwindow#separateKonsoleArguments`
- `internal/termwindow#TestLauncher_OpenRunsTheChosenCommandLine`

**Edit:**
- `internal/termwindow#Launcher`
- `internal/termwindow#resolve`
- `internal/termwindow#TestLauncher_ArgumentsOpenTitledWindowRunningAttach`
- `internal/termwindow#TestResolve_ChoosesLauncherPerPlatform`

**Intent:** Let a caller start Konsole with `--separate` and learn the started process id (D2), or open the window as today (D1).

`Launcher` gains a field `separate`, the arguments of a window run in a process of its own;
`resolve` sets it to `separateKonsoleArguments` on the Konsole launcher alone.
`separateKonsoleArguments` is `konsoleArguments` with `--separate` first.
`IsKonsole` reports whether `separate` is set.

`Open` opens one window and is the one call a placing caller makes:
- with `separate` false it calls `OpenAttach`, today's command line, and returns pid 0;
- with `separate` true it starts the program with the `separate` arguments, detached and reaped like `OpenAttach`,
  logs the spawn with its pid at Info, and returns `cmd.Process.Pid`.
The Linux detach only starts a new session, so that pid is Konsole's own (D2's rationale).
`separate` true on a launcher with no `separate`, which a caller reaches only by skipping `IsKonsole`,
returns a fail-loud wiring error naming the launcher, in `refusal-spec.md`'s sense, which takes no reaching test.
`OpenAttach` and its command line are unchanged.

Tests:
- `TestLauncher_ArgumentsOpenTitledWindowRunningAttach` (`tier0`) gains a `SeparateKonsole` row: `--separate` followed by the Konsole row's arguments.
- `TestResolve_ChoosesLauncherPerPlatform` (`tier0`) also asserts `IsKonsole()` is true exactly on the rows that resolve `konsole`.
  Both rows add under 0.1 seconds to one `tier0` run.
- `TestLauncher_OpenRunsTheChosenCommandLine` is `tier1`, in `termwindow_integration_test.go`.
  It builds a `Launcher` whose program is `sh`, whose `arguments` write `plain` to a marker file and whose `separate` write `separate` to it,
  then `exec sleep 60`; the values are ones the code only passes on.
  Rows: `separate` false writes `plain` and returns pid 0;
  `separate` true writes `separate` and returns a live pid whose parent is the test process (`proc.ParentPID`), which the test ends with `proc.KillPID`.

**ImpactSummary:** Adds two methods and one unexported field to `Launcher`; `OpenAttach` and its callers in `reedcli` and `machinecli` are untouched.
