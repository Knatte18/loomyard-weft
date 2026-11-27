# Card 11 — place-window-child

**Create:**
- `plan:internal/runproc#RolePlacement` -> `const RolePlacement = "placement"`
- `plan:internal/kwinplace#ChildVerbName` -> `const ChildVerbName = "place-window"`
- `plan:internal/kwinplace#Spawn` -> `func Spawn(lymPath, hubPath, worktree, title string, pid int, prime bool) error`
- `plan:internal/reedcli#reedCLI.placeWindowCmd` -> `func (c *reedCLI) placeWindowCmd() *cobra.Command`
- `plan:internal/reedcli#TestPlaceWindowChild` -> `func TestPlaceWindowChild(t *testing.T)`

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

**Intent:** The detached placement child of D10: the one placement call both window paths hand a window to (D12), and the hidden verb it runs.

`Spawn` starts `<lymPath> reed place-window --hub <hubPath> --worktree <worktree> --title <title> --pid <pid>` in the hub,
with `--prime` for the prime's window, detached through `proc.Detach`, reaped through `proc.Reap`,
and logs the spawn with its pid at Info (`PATTERN-child-processes`).
It returns once the child has started, before the window appears in KWin.
For any worktree but the prime it registers the child with
`runproc.Register(fabriclayout.RunProcDir(hubPath), worktree, runproc.ChildRecord(pid, runproc.RolePlacement, args))`,
so the `End` of that worktree's run or teardown sends it SIGTERM; a failed registration only warns.
The prime's child belongs to no run and is not registered.
The verb's name and its flag names are named constants in kwinplace, which the verb reads too.
Callers name `lymPath` through `selfexec.LymBinary`.

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
- runs `kwinplace.NewFlow(runner, afterPlace).Place` under that context,
  with `afterPlace` calling `desktophook.AfterPlacement` with the context it is given, over the worktree's config dirs in its anchor directory (D12).
A resolution that fails still records and settles the window's entry under the told worktree name (`RecordOpen`, then `Settle`), so no child waits on it,
logs a warning naming what failed and its way forward, runs `afterPlace` where the worktree's config dirs resolved, and exits.
A refused `beam.window_placement` names `lym config beam --set window_placement=<value>` in that worktree, or the task's run_settings;
a refused desktop key names card 2's machine-layer set command.
Every warning is logged; the verb always exits zero with an ok envelope.
`reedcli/doc.go`'s sentence on the verbs run by launchers and tmux bindings, not by hand, becomes one naming `place-window` too, run by the window verbs.

What the verb lets through, and its bound (D10's bound):
any local process can run it, but it reads no input but its flags, the hub's places file, the files beside the script lock and KWin;
it moves only the windows of the target desktop and the window whose KWin match (D2) has the told pid, class and title;
it takes the machine-wide lock, so it cannot run beside another placement.

`TestPlaceWindowChild` is `tier1`: it builds `lym` through `lymbin`, makes a hub with `hubforge`,
and sets through `configengine.Set` on the machine layer a `desktop.window_command`
that writes a marker and then waits, at most 30 seconds, for a release file to go, and a `desktop.window_wait_ms` of 600000.
Before it changes `PATH` it starts a `sleep 60` by its absolute path for the window's pid.
It then sets `PATH` to a directory holding links to `sh` and `git` alone, so the child finds no `kscreen-doctor` or `qdbus6`
and its run fails without reaching any desktop; `PATH` is process-global, so the test does not call `t.Parallel` and says so.
Rows, each calling `kwinplace.Spawn`:
- `TaskWindow`: `Spawn` returns while the child runs; the marker appears within a minute, so the window command ran with no wait after the failed run (D12);
  while the command waits, the task's runproc records hold one `RolePlacement` record; the test then removes the release file;
  the places file's entry for the task is settled (D9).
- `PrimeWindow`: the marker appears, and the prime owner holds no `RolePlacement` record.
- `UnknownWorktree`: a `--worktree` naming no worktree of the hub settles that name's entry in the places file, and no marker appears.

**ImpactSummary:** One new hidden reed verb and one new runproc role; `Command`'s verb list and pre-run skip list each gain one entry, and no existing verb changes.
