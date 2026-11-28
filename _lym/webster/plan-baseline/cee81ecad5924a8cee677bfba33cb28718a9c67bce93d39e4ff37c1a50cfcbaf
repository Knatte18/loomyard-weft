# Card 14 — machine-start-placement

**Create:**
- `internal/machinecli#TestStart_HandsEachPlacedWindowToThePlacementChild`

**Edit:**
- `internal/machinecli#deps`
- `internal/machinecli#productionDeps`
- `internal/machinecli#startSession`
- `internal/machinecli#startSession.openWindows`
- `internal/machinecli#newStartDeps`
- `internal/machinecli#startCalls`
- `internal/machinecli#TestStart_WindowAndHubFailuresLeaveTheRestStarted`
- `internal/machinecli#TestStart_PrintsAttachCommandsWhereNoTerminalLauncherResolves`
- `internal/machinecli/doc.go`
- `//docs/overview.md`

**Uses:**
- `internal/kwinplace#OpenWindow`
- `internal/kwinplace#Spawn`

**Intent:** Make `lym machine start` hand each terminal window it opens to the same placement child (D12).

`deps.terminal` returns an opener of the shape `func(dir, title, lymPath string) (int, bool, error)`:
in `productionDeps` it resolves the launcher and opens through `kwinplace.OpenWindow`.
`deps` gains `handPlacement func(lymPath, hubPath, worktree, title string, pid int, prime bool) error`, `kwinplace.Spawn` in production.
`startSession.openTerminal` takes the opener's shape.
`openWindows` hands each window whose checks passed to `handPlacement` with the worktree's name and whether it is the prime;
a hand-off error is a hub warning and the worktree stays started.
A window whose checks failed, or whose hand-off failed, runs no window command, as `lym machine start` runs none today (D12).
`openWindows` still runs after `runBeamStarts`, so a run window may open before its orch's window, which the child's parent wait covers (D8, D12).
`doc.go`'s paragraph on the windows says the same,
and `docs/overview.md`'s paragraph on `lym machine start`'s windows says that under placement they run `desktop.window_command` through the placement child (D12's bound).

Tests, `tier0`:
- `startCalls` gains the hand-offs; `newStartDeps` fakes the opener's new shape, answering `false`, and records hand-offs there;
  the two tests that override `d.terminal` take the new shape and keep their assertions.
- `TestStart_HandsEachPlacedWindowToThePlacementChild`: with the opener answering a pid and `true`, each started worktree's window is handed off once with its pid,
  and the prime's with `prime` true; a failing hand-off is a hub warning and its worktree stays started;
  with the opener answering `false`, nothing is handed off.
It adds under 0.1 seconds to one `tier0` run.

**ImpactSummary:** The `deps` seam's terminal shape changes inside machinecli alone; every other step of `lym machine start` is unchanged.
