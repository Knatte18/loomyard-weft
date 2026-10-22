# Card 6 — The gate of a detached push or board-sync child defers instead of waiting

**Create:**
- `internal/pauserecord#GateCovered`
- `internal/pauserecord#SetGateDeferral`
- `internal/tendercli#DetachedChild`
- `internal/fabricengine#ParentLockedFlag`
- `internal/boardengine#DeferrableSyncFlag`
- `cmd/lyx#detachedChild`
- `cmd/lyx#TestDetachedChild`

**Edit:**
- `internal/pauserecord#Gate`
- `internal/pauserecord#Network`
- `internal/pauserecord#TestNetwork`
- `internal/tendercli#InstallNetworkGate` -> `func InstallNetworkGate(ctx context.Context, pauseSlug *string, child DetachedChild)`
- `internal/tendercli#TestInstallNetworkGate_TellsThePairsSlugAndInstallsNothingOutsideAHub`
- `cmd/lyx#newRoot`
- `internal/pauserecord/doc.go`
- `internal/tendercli/doc.go`

**Uses:**
- `cmd/lyx#pauseSlugOverride`
- `internal/pauserecord#SetGateExempt`

**Intent:**
D6's gate half: a detached push or board-sync child, marked by a hidden flag its spawner sets, never parks as a live process under a pause; every other process keeps the gate's waiting behaviour (`PATTERN-pause-holds-prompts`).
Card 7 makes the spawners set the flags and the children act on the gate.

pauserecord.
`Gate` gains `Defer bool` and `OwnerEntryExempt bool`.
`GateCovered` reports whether a live entry covers the installed gate's process right now, applying exactly the exemptions `Network`'s waiter applies (the parent-lock slug exemption, the exempt entry, `ExemptAll` and `OwnerEntryExempt`); it reports false with no gate installed or an unreadable record.
`SetGateDeferral` stores the function a deferring gate calls and returns the restore of the previous one.
In `Network`, a gate with `Defer` set that `GateCovered` reads covered calls the stored function before any wait; that function is the child's exit (card 7) and does not return, and when none is stored, or it returns, `Network` waits as before, so the gate still fails no call.
`OwnerEntryExempt` lets a live entry of the gate's own slug whose `Caller` is `pauserecord.OwnerCaller` pass; any other slug entry passes only through the live `HoldsParentLock` reading as today, and a hub entry is waited out.
`TestNetwork` gains rows; a deferral row runs `Network` in a goroutine whose stored deferral records its call and ends the goroutine with `runtime.Goexit`, standing in for the child's exit, and the row waits on the goroutine's end:
a `Defer` gate covered by a hub entry calls the deferral once and never calls `Sleep`;
a `Defer` gate with nothing covering calls none and marks its operation in flight;
a `Defer` gate with no deferral stored, and one whose stored deferral returns, waits as before, on the fake `Sleep`, until the entry is lifted;
an `OwnerEntryExempt` gate passes its own slug's owner entry at once, still waits on a hub entry, and still waits on an operator's entry of its own slug, whose `Caller` is not the owner caller, so a slug pause set after the parent lock's release holds it;
`GateCovered` reads each of those states, and reads false with no gate installed and over a record that does not decode.

tendercli and the root.
`DetachedChild` carries `Deferrable`, `ParentLocked` and `HubScope`.
`InstallNetworkGate` sets `Gate.Defer` to `Deferrable && !ParentLocked` and `Gate.OwnerEntryExempt` to `ParentLocked`, so a push child spawned while a parent lock record names its slug never exits for coverage (D6).
With `HubScope` set and a nil `pauseSlug` it resolves the working directory as a worktree root and holds the process under the empty slug, as a non-nil empty `pauseSlug` does, so a board sync is held only by a hub entry whichever worktree spawned it (D2's hub scope); card 7 starts the sync in the board directory.
`detachedChild` reads the executing command's flags: `Deferrable` when `fabricengine.PauseSlugFlag` is set (only a detached push child carries it) or `boardengine.DeferrableSyncFlag` is set, `HubScope` when `boardengine.DeferrableSyncFlag` is set, and `ParentLocked` when `fabricengine.ParentLockedFlag` is set; an undeclared or unset flag reads false.
`newRoot`'s pre-run passes `detachedChild(cmd)` to `InstallNetworkGate`.
The flag names are declared here, beside `PauseSlugFlag` in `fabricengine/spawn.go` and in `boardengine/spawn.go`, and registered on the commands by card 7.
`TestDetachedChild` mirrors `TestPauseSlugOverride`'s table: a set pause-slug flag, a set deferrable flag, a set parent-locked flag beside a pause slug, declared-but-unset and undeclared flags.
`TestInstallNetworkGate_TellsThePairsSlugAndInstallsNothingOutsideAHub` passes a zero `DetachedChild` at every existing call and gains rows: a board-worktree row with `Deferrable` set under a live hub entry, asserting `GateCovered` reads true; one with `ParentLocked` set beside it whose own-slug owner entry reads uncovered; and a `HubScope` row run from a task worktree whose slug alone is paused, asserting `GateCovered` reads false, then true once a hub entry is added.

Bound: only a process whose spawner set one of the two hidden flags defers; the flags are hidden internal flags, and a deferring child pushes nothing more, so a deferral can skip work but never let a write through a pause.
The package docs gain the deferral, the owner-entry exemption and who sets them.

**ImpactSummary:** InstallNetworkGate gains a parameter; its one production caller is newRoot and its test is listed; Network keeps its signature.

**Commit:** 6: the gate of a detached push or board sync defers under a pause
