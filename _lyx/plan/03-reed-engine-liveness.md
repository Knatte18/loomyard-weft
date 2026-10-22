# Card 3 — Reed reads a shell-only strand as dead

**Edit:**
- `internal/reedengine#Engine`
- `internal/reedengine#New`
- `internal/reedengine#Engine.Status`
- `internal/reedengine#directoryRows` -> `func directoryRows(strands []Strand, live []LivePane, alive map[string]bool) []DirectoryRow`
- `internal/reedengine#Engine.Directory`
- `internal/reedengine#Engine.Resume`
- `internal/reedengine#Engine.AddStrandUnless`
- `internal/reedengine#classifyIfAbsent`
- `internal/reedengine#Engine.launchStrandLocked`
- `internal/reedengine#TestStatus_ReportsOwnWindowStrandsFromTheSessionListing`
- `internal/reedengine#TestDirectory_MapsEachStrandToItsRow`
- `internal/reedengine#TestAddStrand_IfAbsent_NoOps`
- `internal/reedengine#TestOwnWindow_RealTmux`
- `internal/reedengine/doc.go`
- `internal/shuttleengine/claudeengine/template.yaml`

**Create:**
- `plan:internal/reedengine#launchGrace` -> `const launchGrace = 30 * time.Second`
- `plan:internal/reedengine#shellBaseName` -> `func shellBaseName(command string) string`
- `plan:internal/reedengine#engineAliveIDs` -> `func engineAliveIDs(present []LivePane, age func(pid int) (time.Duration, bool), checkShell bool) map[string]bool`
- `plan:internal/reedengine#Engine.engineAliveIDSet` -> `func (e *Engine) engineAliveIDSet(present []LivePane) map[string]bool`
- `plan:internal/reedengine#TestEngineAliveIDs` -> `func TestEngineAliveIDs(t *testing.T)`

**Uses:**
- `internal/reedengine#aliveIDSet`
- `internal/reedengine#presentPanes`
- `plan:internal/reedengine#paneProcessAge`
- `internal/reedengine#planResumeLaunches`
- `internal/reedengine#newTestEngine`
- `internal/reedengine#fakeTmux`

**Intent:**
D1: reed judges a strand live only when its pane is listed and not dead, sits in the reed session (already true: `presentPanes` reads only this session's listings), and its foreground command is not the shell the pane was started with.

`engineAliveIDs` in `internal/reedengine/apply.go`, beside `aliveIDSet`, is the one strand liveness predicate.
It returns the ids of the panes in `present` that are not dead, minus every shell-only pane past the grace:
when `checkShell` is true, `StartCommand` and `CurrentCommand` are both non-empty, `shellBaseName` of the two is equal, and `age(PID)` answers true with an age of at least `launchGrace`, the pane is left out.
An empty start command (a pane spawned with no `default-command` pin) or an empty foreground command skips condition 3 and reads live; an age `age` cannot read counts as inside the grace, so a failed read never reads a pane dead.
`age` is called only for a pane whose commands match, so a running engine costs no `/proc` read.
`shellBaseName` trims spaces and one pair of surrounding `"` or `'`, takes the first space-separated word, its `filepath.Base`, and drops one leading `-`, so the login shell `-bash` and the pin `/bin/bash` (or `"/usr/bin/zsh -l"`) compare equal.
`Engine.engineAliveIDSet` calls `engineAliveIDs(present, e.paneAge, runtime.GOOS != "windows")`, so condition 3 is skipped on Windows (a PowerShell cmdlet runs inside the shell).
`Engine` gains the seam field `paneAge func(pid int) (time.Duration, bool)`, which `New` sets to `paneProcessAge`; a test sets it to a stub.
A nil `paneAge`, on an `Engine` built by hand rather than through `New`, reads as "age unknown, inside the grace": `engineAliveIDSet` passes a func answering false, so condition 3 never reads such a pane dead.

Readers moved from `aliveIDSet` to `engineAliveIDSet` (D1's Readers):
- `Engine.Status`: `StrandStatus.Live` reads `engineAliveIDSet(present)`. `Present`, `OwnWindow` and `PanePID` stay pane-level: `PanePID` is still the listed pid of any pane that is not dead, since creel maps processes to strands through it, and `Present` is what shuttle's stop and the died-window removal decide on, so `strandRemovable` (internal/shuttleengine/stop.go) still removes a shell-only own-window strand with no change there. Through `Status` this reaches `lyx orch status`, `lyx orch start`, webster's `StrandLive`, loom's driver check, creel's `Strand.Live` and shuttle's wait and attach with no change in those packages.
- `directoryRows` takes the alive set as a new parameter and sets `Live`, `Title` and `Drift` only for a strand in it; `Engine.Directory` passes `e.engineAliveIDSet(present)` and the cold path passes nil. Drift stays whatever `planTitleRepairs` repairs, which reads pane-level aliveness.
- `Engine.Resume` passes `engineAliveIDSet(presentPanes(...))` to `planResumeLaunches`, so a shell-only strand past the grace is relaunched and its old pane is killed by `killReplacedOwnWindowPaneLocked`, as D1's Consequences state.
- `Engine.AddStrandUnless`'s `--if-absent` path passes the same set to `classifyIfAbsent`, so a shell-only match is relaunched rather than answered as a live no-op.
Pane-level checks keep `aliveIDSet` (conditions 1 and 2 only): the Selvage pane's health (`newReapPolicy`), pane-option reassertion (paneoptions.go) and title repair (namerepair.go), since Selvage is a shell by design; none of them changes.
The doc comments of `classifyIfAbsent` and `Engine.launchStrandLocked`, and the inline comments in `Engine.Status` and `Engine.Resume`, which say the set is `aliveIDSet` and that it excludes only dead-but-present panes, are rewritten to name the engine judgment and its shell-only case.

`internal/shuttleengine/claudeengine/template.yaml`: the `binary` key's comment gains that the binary must be the engine or `exec` it, since a wrapper that does not `exec` leaves its shell in the foreground and reed reads the strand dead (Constraints).
`internal/reedengine/doc.go` gains the liveness rule: the three conditions, the 30-second grace read from the pane's first process, the empty-command and unreadable-age skips, the Windows skip, which readers take it and which pane-level checks do not, and the false-live and false-dead residuals of D1's Bound.

Tests:
- `TestEngineAliveIDs` is a table over `engineAliveIDs` with a stub `age`: an engine in the foreground (`claude` over start `/bin/bash`) is alive; a shell-only pane (`bash` over `/bin/bash`) aged 31s is not; aged 29s is; one whose age the stub cannot read is; one with an empty start command is; one with an empty current command is; `-bash` over `"/bin/bash"` aged 31s is not; a shell-only pane aged 31s with `checkShell` false is (the Windows skip); a dead pane is absent whatever its commands; and the stub records that `age` was called only for the matching panes.
- `TestStatus_ReportsOwnWindowStrandsFromTheSessionListing` sets `e.paneAge` to a stub answering 31s for one pid, scripts its window listing line with tail `t\x1fbash\x1f/bin/bash` and adds rows: that strand reads `live: false, present: true, own: true` with its `PanePID` still the listed pid, and a strand whose tail is `t\x1fclaude\x1f/bin/bash` reads live.
  It keeps every existing row unchanged, since a tail with no separator reads both commands empty.
- `TestDirectory_MapsEachStrandToItsRow` passes the alive set to `directoryRows` and gains a row whose pane is present and not dead but missing from the alive set, which reads dormant with an empty title.
- `TestEngineAliveIDs` gains a row driving `Engine.engineAliveIDSet` on an `Engine` literal with a nil `paneAge`: a shell-only pane reads alive.
- `TestAddStrand_IfAbsent_NoOps` gains a row whose name-matched own-window strand's pane lists `bash` over `/bin/bash` with `e.paneAge` stubbed to 31s: the add is no no-op, it runs `split-window` and returns the strand bound to the new pane, so a reverted `classifyIfAbsent` set fails it.
- `TestOwnWindow_RealTmux` (tier 2, a real tmux) gains a subtest that pins the real listing and the Resume reader: an own-window strand added with an empty `Cmd` holds only its shell; with `e.paneAge` stubbed to 31s, `Status` reads it `live: false, present: true`, and `Resume` relaunches it onto a new pane id and kills the old pane, while the Selvage pane, a shell by design, keeps its pane id, so a Selvage health check moved to the engine judgment fails it.
  With the stub answering 1s, `Status` reads the same strand live.

**ImpactSummary:** One predicate replaces aliveIDSet at the four strand-liveness readers inside reedengine; every consumer outside it (orch, webster, loom, creel, shuttle) gets the change through `StrandStatus.Live` with no code change, and `directoryRows` has its one production caller on this card.

**Commit:** 3: reed reads a strand whose engine exited to the shell as dead
