# Card 12 — place-window-child

**Create:**
- `internal/reedcli/placewindowchild_integration_test.go`
- `internal/runproc#RolePlacement`
- `internal/kwinplace#ChildVerbName`
- `internal/kwinplace#Spawn`
- `internal/reedcli#reedCLI.placeWindowCmd`
- `internal/reedcli#TestPlaceWindowChild`

**Edit:**
- `internal/reedcli/cli.go#`
- `internal/reedcli/doc.go`

**Extends:**
- `internal/kwinplace/doc.go`

**Uses:**
- `internal/runproc/doc.go`
- `internal/runproc#Register`
- `internal/runproc#ChildRecord`
- `internal/fabriclayout#RunProcDir`
- `internal/fabriclayout#HubLogsDir`
- `internal/hubgeom#ParentOf`
- `internal/hubgeom#TaskSettingChain`
- `internal/dobbyengine#LookupWindowPlacement`
- `internal/desktophook#LoadConfig`
- `internal/desktophook#AfterPlacement`
- `internal/fabricengine#PrimeName`
- `internal/fabricengine#ConfigDirs`
- `internal/fabricengine#List`
- `internal/lymcwd#NamedWorktree`
- `internal/kwinplace#Mode`
- `internal/kwinplace#NewRunner`
- `internal/pauserecord#RunProcessEnv`
- `internal/pauserecord#RunProcessMarked`

**Intent:** The detached placement child of D10: the one placement call both window paths hand a window to (D12), and the hidden verb it runs.

`Spawn` starts `<lymPath> reed place-window --hub <hubPath> --worktree <worktree> --title <title> --pid <pid>` in the hub,
with `--prime` for the prime's window, detached through `proc.Detach`, reaped through `proc.Reap`,
and logs the spawn with its pid at Info (`PATTERN-child-processes`).
It returns once the child has started, before the window appears in KWin.
It registers the child only when the worktree is not the prime
and the calling process's environment holds a non-empty `pauserecord.RunProcessEnv`, which a run command sets for itself and every process it starts (D10).
It registers it with `runproc.Register(fabriclayout.RunProcDir(hubPath), worktree, runproc.ChildRecord(pid, runproc.RolePlacement, args))`,
so the `End` of that worktree's run or teardown sends it SIGTERM; a failed registration only warns.
A child opened outside a run, such as by `lym machine start` or by a `lym reed window` run by hand in a pane,
belongs to no run, is not registered and ends within its own bounds.
The prime's child belongs to no run and is not registered.
The verb's name and its flag names are named constants in kwinplace, which the verb reads too.
Callers name `lymPath` through `selfexec.LymBinary`.

A `RolePlacement` record lives as long as its child: at most the 10-second window wait, the 180-second parent wait and the 120-second lock wait plus its script bounds,
then the window command's 30-second bound and its 5-second group kill (card 4).
Every reader of a worktree's live runproc records that does not pick records by role counts it for that time, and each is right to:
- `internal/pairteardown/leftover.go`'s `requireStillOrphan` refuses to remove an orphan pair while it lives:
  a window was just opened on that pair, so the pair is in use, and the refusal clears once the child exits;
- `internal/battencli/leftoverpairs.go`'s orphan view reports `ProcessLive` for the pair, which is true;
- `internal/battencli/cancel.go`'s `otherLeft` counts it as something left, so the cancel runs and its `End` sends the child SIGTERM,
  where without it the cancel would refuse as if nothing were left;
- `internal/pairteardown/teardown.go`'s `ownedProcesses` names it in a refusal's list of live owned processes,
  and its role checks ask only for `RoleBatten` and `RoleDobby`, so a placement record changes no teardown decision.
The other readers pick records by another role and pass over a placement record:
`internal/fabriccli/anchordeps.go`'s `BattenLive`, `DobbyPID` and `dobbyPassInFlight`, `internal/battencli/wire.go`'s `gateProcessLive`
and `internal/pairteardown/teardown.go`'s `driverRecordPID`.
No reader therefore gets a role check.

`placeWindowCmd` is `lym reed place-window`, hidden, with the `internal` audience, its four value flags required.
`Command` adds it, and its pre-run skips it like `window`, since it resolves the hub from its flag;
`cli.go`'s file comment and `Command`'s pre-run comment name it among the verbs that skip the resolution.
Its `RunE`:
- points the durable log sink at `fabriclayout.HubLogsDir(hub)` and takes a context that SIGTERM cancels;
- resolves the worktree's location with `lymcwd.NamedWorktree`, and the prime's worktree name through `fabricengine.PrimeName`;
- resolves the parent: none for the prime's own window;
  else `hubgeom.ParentOf`, whose `""` names the prime, which the verb turns into the prime's worktree name for the `Request` (D8);
- resolves the mode with `kwinplace.Mode`: `dobbyengine.LookupWindowPlacement` over `hubgeom.TaskSettingChain`,
  else `desktop.window_placement` from `desktophook.LoadConfig` over the worktree's config dirs, with the two tile sizes from the same load (D7);
- lists the hub's worktrees through `fabricengine.List`;
- makes the runner with `kwinplace.NewRunner`;
- runs `kwinplace.NewFlow(runner, afterPlace).Place` under that context,
  with `afterPlace` calling `desktophook.AfterPlacement` with the context it is given, over the worktree's config dirs in its anchor directory (D12).
A resolution that fails, a failed `kwinplace.NewRunner` among them, still records and settles the window's entry under the told worktree name (`RecordOpen`, then `Settle`),
so no child waits on it, logs a warning naming what failed and its way forward, runs `afterPlace` where the worktree's config dirs resolved, and exits.
A refused `beam.window_placement` names `lym config beam --set window_placement=<value>` in that worktree, or the task's run_settings;
a refused desktop key names card 3's machine-layer set command.
Every warning is logged; the verb always exits zero with an ok envelope.
`reedcli/doc.go`'s sentence on the verbs run by launchers and tmux bindings, not by hand, becomes one naming `place-window` too, run by the window verbs.

What the verb lets through, and its bound (D10's bound):
any local process can run it, but it reads no input but its flags, the hub's places file, the files beside the script lock and KWin;
it moves only the windows of the target desktop and the window whose KWin match (D2) has the told pid, class and title;
it writes only the told hub's places file, which `RecordOpen` creates under `<hub>/_bolt/.lym/window-places/`, the hub's log files,
the files beside the per-user script lock, and each script's temp file under `lymtemp`'s directory;
it runs only the `desktop.window_command` the told worktree's own config stack names, which the same user could run directly;
it takes the machine-wide lock, so it cannot run beside another placement.

`TestPlaceWindowChild`, in `internal/reedcli/placewindowchild_integration_test.go`, is `tier1`.
It builds `lym` through `lymbin`, makes a hub with `hubforge`,
and sets through `configengine.Set` on the machine layer a `desktop.window_command`
that writes a marker and then waits, at most 30 seconds, for a release file to go, and a `desktop.window_wait_ms` of 600000.
Before it changes `PATH` it starts a `sleep 60` by its absolute path for the window's pid.
It then sets `PATH` to a directory holding links to `sh` and `git` alone, so the child finds no `kscreen-doctor` or `qdbus6`
and its run fails without reaching any desktop.
Every row sets `pauserecord.RunProcessEnv` itself with `t.Setenv`, so the mark of the run that runs the test reaches no row;
`PATH` and that variable are process-global, so the test does not call `t.Parallel` and says so.
Rows, each calling `kwinplace.Spawn`:
- `TaskWindow`, with the variable set to `pauserecord.RunProcessMarked`: `Spawn` returns while the child runs; the marker appears within a minute, so the window command ran with no wait after the failed run (D12);
  while the command waits, the task's runproc records hold one `RolePlacement` record; the test then removes the release file;
  the places file's entry for the task is settled (D9).
- `PrimeWindow`, with the variable marked: the marker appears, and the prime owner holds no `RolePlacement` record.
- `UnmarkedTaskWindow`, on a task worktree of its own, with the variable set empty: the marker appears;
  while the command waits, the task's runproc records hold no `RolePlacement` record; the test then removes the release file.
- `RefusedTaskMode`, on a task worktree of its own, so no other row resolves its mode, with the variable set empty: that worktree's beam config file holds `window_placement: sideways`;
  the task's places entry is settled, the marker appears, and the child's log under `fabriclayout.HubLogsDir` holds a warning naming `lym config beam --set window_placement=<value>`.
- `RefusedMachineMode`, on a hub of its own, with the variable set empty: `configengine.Set` on the machine layer sets `desktop.window_placement` to `sideways`;
  the task's places entry is settled, and the child's log holds a warning naming card 3's machine-layer set command;
  no marker appears, since the window command's own load of the desktop config refuses the same key.

**ImpactSummary:** One new hidden reed verb and one new runproc role; `Command`'s verb list and pre-run skip list each gain one entry, and no existing verb changes.
