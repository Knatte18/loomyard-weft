---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/kwinplace/snapshot.go#Snapshot.ZoneOf
    summary: "Snapshot.ZoneOf is called only by a test, against PATTERN-test-only-code"
    fix: "Delete Snapshot.ZoneOf. In internal/kwinplace/recordings_test.go (package kwinplace) call zoneHolding(snapshot.Tiles[gridDesktop], \"{7028d260-eabc-4af0-b043-3833ac29ac27}\") in its place. Card 6 names the method; the constraint wins."
    origin: lens:code-affected-set
  - id: F2
    severity: BLOCKING
    class: consistency
    location: internal/desktophook/config.go#LoadConfig
    summary: "The new desktop keys are repeated string literals handed to configengine.KeyName, where PATTERN-config-key-names asks for named constants"
    fix: "In internal/desktophook/config.go declare one constant each for \"window_placement\", \"tiling.min_tile_width\" and \"tiling.min_tile_height\". Use them in ConfigSpec's Categories, in the KeyName calls, in the tile loop and in the --set text of both refusals."
    origin: lens:code-affected-set
  - id: F3
    severity: BLOCKING
    class: consistency
    location: internal/kwinplace/doc.go
    summary: "The package doc states a rule other packages must follow (no other package reaches KWin or kscreen), which PATTERN-code-conventions bars from doc.go"
    fix: "In internal/kwinplace/doc.go end the second package-comment sentence at: It owns the KWin and kscreen boundary and the placement flow. Drop the clause saying no other package reaches either."
    origin: lens:code-affected-set
  - id: F4
    severity: BLOCKING
    class: design
    location: internal/kwinplace/contract_test.go#TestContract_TheRealKWinAppliesAsRecorded
    summary: "The tier3 contract tests do not compare what lym reads from a real apply with the recording, against PATTERN-test-stand-ins and card 8"
    fix: "In the apply test place on a 1 by 2 grid (Cols 2, Zones [placed, empty]), since a 1 by 1 grid has no leaves to read. Run the script through runner.runScript with ApplyScript, compare lineKinds of those lines with lineKinds of ReportLines over neve-apply/journal.txt for applyRunID, parse them with ParseApplyReport, and assert two tiles with geometry and padding and the placed window in zone 0, tiled. In the read-only test also compare the first loadScript answer with neve-qdbus/load.txt by kind (a non-negative integer) and assert the window fields (pid, resource class, caption, frame) and tile fields the snapshot carries are filled. Running tier3 is the operator's call, so the changed tests are compiled only."
    origin: lens:code-tests
  - id: F26
    severity: BLOCKING
    class: decision
    location: internal/machinecli/start_test.go#newStartDeps
    summary: "The machinecli tests hand-write a stand-in for kwinplace.Spawn (and OpenWindow's answer), which PATTERN-test-stand-ins forbids; card 14 asks for exactly that"
    fix: "Two choices for the author. (a) Accept the gap: the parent rules that card 14's seam fakes stand, recorded where the standing fakes are listed (docs/research/no-fake-other-modules.md). (b) Build what is missing: drive real kwinplace.Spawn from the machinecli tests through a recording wrapper that passes each call on, which needs a tier1 test with a lymbin-built lym and a hub fixture, and an opener that can answer placing true without a real Konsole, which does not exist today. Unverified: whether machinecli's fixtures can be rebuilt on hubforge for this."
    origin: lens:code-affected-set
  - id: F5
    severity: MEDIUM
    class: design
    location: internal/kwinplace/places.go#updatePlaces
    summary: "The worktree list is read when the child starts and applied minutes later, so entries of worktrees created meanwhile are deleted and later recreated with no title"
    fix: "Read the hub's worktree names inside the places write: have Request carry a function returning them (filled in internal/reedcli/placewindowchild.go from fabricengine.List) and call it within RecordReport's mutate, under places.lock, so every entry present belongs to a worktree that existed before the list was read. On a listing error drop no entry. Add a row that records an entry for a worktree added after the request was built and asserts it survives the report with its title."
    origin: lens:code-correctness
  - id: F6
    severity: MEDIUM
    class: design
    location: internal/kwinplace/places.go#Settle
    summary: "A child of an older window overwrites the entry of the worktree's newer window, and can make a settled entry unsettled again"
    fix: "In Settle leave an existing entry unchanged when entry.PID differs from pid. In updatePlaces change the placed entry only when its PID is 0 or equals update.PID; keep the rules for the other windows. Add one TestPlacesFile or TestUpdatePlaces row: RecordOpen pid 2 after pid 1, then a settle and a report for pid 1 leave the entry as pid 2 left it."
    origin: lens:code-correctness
  - id: F7
    severity: MEDIUM
    class: design
    location: internal/kwinplace/places.go#updatePlaces
    summary: "A window whose home desktop no longer exists keeps the dead home id and never gets a new one"
    fix: "Add a HomeGone field to ReportUpdate, set in reportUpdateFrom (flow.go) when the placed entry's Home is non-empty and no desktop of snap.Desktops has that id. In updatePlaces rehome the placed window when Home is empty or HomeGone, still excluding Fallback and Overflowed. Add a TestUpdatePlaces row for a gone home."
    origin: lens:code-correctness
  - id: F8
    severity: MEDIUM
    class: design
    location: internal/reedcli/window.go#handTerminalWindow
    summary: "A relative directory argument yields a relative hub path, so the child works on the hub's parent and neither places the window nor runs the window command"
    fix: "In handTerminalWindow make dir absolute with filepath.Abs before lymcwd.ResolveWorktree, returning the error. Cover it with the in-worktree test of F16, passing a relative dir or one holding a .. element."
    origin: lens:code-correctness
  - id: F9
    severity: MEDIUM
    class: design
    location: internal/reedcli/window.go#handTerminalWindow
    summary: "A git checkout that is in no hub is handed to the child, which creates a _bolt/.lym tree and logs in the checkout's parent directory"
    fix: "After resolving the worktree, return nil with the same Debug log when fabricengine.ConfigDirs(l).Board is empty (no Bolt checkout under the parent), before IsPrimeWorktree and Spawn. Add a row to TestHandTerminalWindow_DirectoryInNoWorktreeHandsNothing for a plain git repository, asserting no Spawn and no _bolt directory beside it."
    origin: lens:code-correctness
  - id: F10
    severity: MEDIUM
    class: design
    location: internal/desktophook/window.go#runWindowCommand
    summary: "lym reed window and lym ide now run the window command in its own process group, so a Ctrl-C or group kill of the verb leaves it running unbounded, unlike before"
    fix: "Configure the process group (proc.ConfigureGroupTerminate) only for the AfterPlacement path, for example by a field or parameter afterPlacement sets and afterWindow does not; AfterWindow keeps the plain exec.CommandContext kill with WaitDelay. TestAfterWindow's timeout row must still pass. Card 4 words the group as the shared runner's, while also saying AfterWindow keeps its behaviour, which D1 and D12 ask."
    origin: lens:code-correctness
  - id: F11
    severity: MEDIUM
    class: consistency
    location: cmd/lym/hermeticenv_test.go#allowedNonHermetic
    summary: "The diff widens the hermetic-git guard's allowlist with no Loosened-Check trailer naming a measured cause"
    fix: "Remove the internal/kwinplace/contract_test.go entry. In internal/kwinplace/testmain_test.go call gitkit.HermeticGitEnv() before os.Exit(tmuxkit.Main(m)), as internal/reedcli/testmain_test.go does, which satisfies the guard without an exemption, and update that file's header comment."
    origin: handler
  - id: F12
    severity: MEDIUM
    class: design
    location: internal/kwinplace/flow_test.go
    summary: "No test fails if reportUpdateFrom ignores the apply report or records only the placed window"
    fix: "In ReopeningUnderAPanelKeepsEveryKey set Stacking on the three windows to an order different from the zone order, so keys survive only when the report's tiles are read. Add an assertion, in that row or one beside it, that another settled worktree's window moved by hand is rehomed in the places file after a placement."
    origin: lens:code-tests
  - id: F13
    severity: MEDIUM
    class: design
    location: internal/reedcli/placewindowchild_integration_test.go
    summary: "Nothing asserts what placementRequest resolves: parent, prime slug, mode precedence, tile sizes and worktree names"
    fix: "Add a tier1 test in this package over a hubforge hub with a task pair, which calls placementRequest for the prime's window and a task's and asserts Parent (empty for the prime, the prime's name for a task under it), Mode (per-task value over the machine's), the two tile sizes and the worktree names (or the listing function F5 introduces)."
    origin: lens:code-tests
  - id: F14
    severity: MEDIUM
    class: design
    location: internal/reedcli/placewindowchild_integration_test.go
    summary: "No test signals the placement child or cancels it during a wait, so D12's SIGTERM behaviour is only partly held"
    fix: "Add to TestPlaceWindowChild a held row that ends the task's registered child with runproc.End(fabriclayout.RunProcDir(hubPath), owner, os.Getpid()) and asserts the entry is settled and the window command's process is gone while its release file still exists. In TestFlow_Place add a row cancelling from onSleep during a wait, asserting a settled entry and no after-place call. In TestAcquireWithin add a cancelled-context case."
    origin: lens:code-tests
  - id: F15
    severity: MEDIUM
    class: design
    location: internal/kwinplace/recordings_test.go
    summary: "D4's lowest-priority rule is unpinned, since a reader returning the first enabled output passes both recordings"
    fix: "In the Kscreen subtest, reverse the recorded outputs array (reordering a recording is allowed), marshal it as the existing cut does, and assert TilingOutput still returns DP-2."
    origin: lens:code-tests
  - id: F16
    severity: LOW
    class: design
    location: internal/reedcli/handterminalwindow_integration_test.go
    summary: "handTerminalWindow's in-worktree path (prime detection and what it passes to Spawn) has no test"
    fix: "Add a tier1 test that calls handTerminalWindow with a lymbin-built lym on a hubforge pair's worktree, with PATH holding no desktop tool as TestPlaceWindowChild arranges, and asserts the places entry of that worktree gets the title and pid. A row of TestPlaceWindowChild calling handTerminalWindow in place of kwinplace.Spawn serves."
    origin: lens:code-tests
  - id: F17
    severity: LOW
    class: design
    location: internal/kwinplace/planner.go#Anchor
    summary: "A settled parent with no usable desktop (empty or removed last desktop, no live window) anchors on the current desktop without the fallback mark, so the child records it as home"
    fix: "In placeLocked (flow.go), when Anchor reports known but its desktop is not among snap.Desktops, set anchor to snap.CurrentDesktop and fallback to true. Add a TestFlow_Place row with a settled parent entry holding no last desktop, asserting the window takes no home."
    origin: lens:code-correctness
  - id: F18
    severity: LOW
    class: design
    location: internal/kwinplace/flow.go#reportUpdateFrom
    summary: "A minimized window homed on the target is released by the grid rebuild, ordered last and has its order key rewritten though nobody moved it"
    fix: "Mark minimized windows in SeenWindow and leave them out of the sitting list in updatePlaces, so a renumber treats them as absent and keeps their position. Set the mark in reportUpdateFrom from the snapshot's Minimized. Add a TestUpdatePlaces row with keys 0 (minimized), 1 and 2."
    origin: lens:code-correctness
  - id: F19
    severity: LOW
    class: design
    location: internal/kwinplace/apply.js#buildGrid
    summary: "One-row grids set the columns' relativeGeometry once, where the reference set-tiling-grid.sh D6 names sets it a second time"
    fix: "In buildGrid, when rows is 1 and there is more than one column, set each column's relativeGeometry a second time after the first pass, as the else branch of ~/.local/bin/set-tiling-grid.sh does. Unverified: whether one pass leaves 3 to 5 columns uneven on KWin; nobody drove it, and the one-row recording has 2 columns."
    origin: lens:code-correctness
  - id: F20
    severity: LOW
    class: design
    location: internal/kwinplace/planner_test.go#TestPlan
    summary: "next from the last desktop and an anchor naming no desktop have no row"
    fix: "Add two TestPlan rows: mode next with anchor d3 and no home yields no target and the warning every desktop from B2 is full; mode same with anchor empty and no home lands on the current desktop d1."
    origin: lens:code-tests
  - id: F21
    severity: LOW
    class: design
    location: internal/kwinplace/runner_test.go#TestRunner_ScriptRuns
    summary: "Re-reading the journal and isScriptLoaded, the journalctl and script-object arguments, and the tiling-output check of the shared report are unpinned"
    fix: "Give recordedExec options to serve a journal without the end marker on its first read and loaded-true on the first isScriptLoaded after an unload, and add rows that succeed on the second read. Have it assert the journalctl arguments (--user, the unit, --since) and the /Scripting/Script<id> path of run. Add a row querying another output name after a cached DP-2 report and assert two loads."
    origin: lens:code-tests
  - id: F22
    severity: LOW
    class: design
    location: internal/kwinplace/runner_test.go#TestRunner_OverlappingRunsTakeTheScriptLockInTurn
    summary: "The overlap tests force no overlap, so they can pass with the script lock or the placement lock broken"
    fix: "Add to TestRunner_ScriptRuns a case that holds the script lock and calls Apply with a short scriptLockWait, asserting errScriptLockBusy and no load. In TwoFlowsApplyInTurn assert inside the first apply that lock.TryAcquireWriteLock on the placement lock path is refused."
    origin: lens:code-tests
  - id: F23
    severity: LOW
    class: consistency
    location: pattern/PATTERN-test-speed.md
    summary: "Two statements still call tier3 LLM-only after the diff widens it to desktop sessions"
    fix: "In pattern/PATTERN-test-speed.md, section No gate runs tier3, say a tier3 run needs a live LLM or desktop session and that an LLM run is billed and nondeterministic. In contracts/specs/loom-plan-spec.md (line 346) say tier 3 tests drive a real LLM or a real desktop session."
    origin: lens:code-affected-set
  - id: F24
    severity: NIT
    class: design
    location: internal/kwinplace/runner.go#Runner.unload
    summary: "An unreachable KWin or qdbus6 with a recorded leftover is reported as a script still loaded, and the real error is dropped"
    fix: "Have unload keep the last isScriptLoaded error and have LeftoverScriptError carry and print it (with Unwrap), saying the load state could not be read when it is set; keep the by-hand unload command."
    origin: lens:code-correctness
  - id: F25
    severity: NIT
    class: design
    location: internal/desktophook/config_test.go#TestLoadConfig
    summary: "The tile-height refusal, the report parse errors and Place's joining of the after-place error have no coverage"
    fix: "Add a ZeroTileHeightRefused row to desktophook TestLoadConfig; a case in internal/kwinplace/recordings_test.go or a report test feeding ParseQueryReport an unknown kind; and a TestFlow_Place row whose afterPlace returns an error that Place's result must wrap."
    origin: lens:code-tests
---

### [BLOCKING] Snapshot.ZoneOf is test-only code

**Location:** `internal/kwinplace/snapshot.go:72`
**Issue:** The one caller of `ZoneOf` is `recordings_test.go:138`; production reaches `zoneHolding` through `CurrentOrder`. `PATTERN-test-only-code` bars a production function no `main` reaches. Its guard runs only in upkeep, so no task gate reports it.

### [BLOCKING] New config keys are literals, not named constants

**Location:** `internal/desktophook/config.go`
**Issue:** `"window_placement"` and the two tile keys are typed in `ConfigSpec`, in the `KeyName` calls and in the `--set` text. `PATTERN-config-key-names` asks for the helper to render from named constants.

### [BLOCKING] doc.go carries a rule for other packages

**Location:** `internal/kwinplace/doc.go`
**Issue:** "no other package reaches either" binds code outside the package. `PATTERN-code-conventions` says such a rule is a PATTERN entry, never a `doc.go` paragraph, and the task declares no PATTERN change.

### [BLOCKING] The apply contract test does not compare with the recording

**Location:** `internal/kwinplace/contract_test.go`
**Issue:** The only use of `neve-apply` is that its usable area is non-zero. Line kinds, tiles, zone and tiled are never compared, and the 1 by 1 grid it builds has no leaves, so none could be. `PATTERN-test-stand-ins` asks the contract test to compare fields and kinds, and card 8 asks it by name.

### [BLOCKING] machinecli tests stand in for kwinplace

**Location:** `internal/machinecli/start_test.go`
**Issue:** `productionDeps` fills `handPlacement` with `kwinplace.Spawn`, and the tests replace it with a recorder that returns a scripted error. `PATTERN-test-stand-ins` names that a stand-in. No fix was found that keeps card 14's tier0 coverage without it, so the choice is the author's.

### [MEDIUM] A stale worktree list deletes live entries

**Location:** `internal/kwinplace/places.go` (`updatePlaces`), `internal/reedcli/placewindowchild.go` (`placementRequest`)
**Issue:** A child lists the worktrees at start and records after up to the window, parent and lock waits. A pair created and opened in that window has its entry removed, losing home and order key. Its own later `Settle` or report recreates the entry without a `Title`, and `Window.Matches` then accepts any caption of that pid. A parent entry removed this way sends its children into the full parent wait. D9 drops only worktrees the hub no longer has.

### [MEDIUM] An older child overwrites a newer window's entry

**Location:** `internal/kwinplace/places.go` (`Settle`, `updatePlaces`)
**Issue:** Neither compares its pid with `entry.PID`. Window A waits, the worktree is reopened as B, B finishes, A finishes: `SettledBy` is A's pid against B's `PID`, so the entry reads unsettled, and `LastDesktop` and possibly `Home` are A's.

### [MEDIUM] A removed home desktop is never replaced

**Location:** `internal/kwinplace/places.go` (`updatePlaces`)
**Issue:** `Plan` treats a home absent from the snapshot as no home, but `rehome` runs only for `Home == ""`. Every later open is placed by mode, waits for the parent and keeps no order. D8 calls this a window without a home, and D9 gives such a window the desktop it lands on.

### [MEDIUM] A relative directory breaks the hand-off

**Location:** `internal/reedcli/window.go` (`handTerminalWindow`)
**Issue:** `lymcwd.ResolveWorktree` keeps a relative path relative. `lym reed window ../task` yields hub `..`, which `Spawn` uses both as `cmd.Dir` and as `--hub`, so the child resolves one level too high. For `.` the worktree name is `.`. No window command runs, which it did before this change.

### [MEDIUM] A checkout outside any hub gets a _bolt tree beside it

**Location:** `internal/reedcli/window.go` (`handTerminalWindow`)
**Issue:** Only a resolve error counts as "no hub". For a plain repository, `HubPath` is its parent directory, and the child creates `_bolt/.lym/window-places` and log files there. Card 13 says nothing is handed where there is no hub.

### [MEDIUM] AfterWindow's callers changed behaviour

**Location:** `internal/desktophook/window.go` (`runWindowCommand`)
**Issue:** `ConfigureGroupTerminate` sets `Setpgid` for every caller. `lym reed window` and `lym ide` handle no signals, so a Ctrl-C or a kill of their process group now ends `lym` and leaves the command running with no bound. D1 and D12 ask for today's behaviour when the checks fail. Reasoned from `procgroup_linux.go`, not driven.

### [MEDIUM] Hermetic-git guard widened without a trailer

**Location:** `cmd/lym/hermeticenv_test.go`
**Issue:** Commit `d168c0b36` adds an allowlist entry to a guard that exists at the review base. No commit of the range carries `Loosened-Check:`, and the commit's reason is no measurement. The exemption is avoidable. The `unboundedWaitFiles` addition tightens its guard and is fine.

### [MEDIUM] reportUpdateFrom's use of the apply report is unpinned

**Location:** `internal/kwinplace/flow_test.go`
**Issue:** In the panel row the order keys equal the stacking order, so dropping `report.Tiles` gives the same result. No flow row asserts another worktree's entry changes.

### [MEDIUM] The verb's resolution is unasserted

**Location:** `internal/reedcli/placewindowchild_integration_test.go`
**Issue:** Every row fails at the tiling-screen read, so a wrong parent, mode, tile size or worktree list passes every test.

### [MEDIUM] SIGTERM behaviour is partly unheld

**Location:** `internal/reedcli/placewindowchild_integration_test.go`, `internal/kwinplace/flow_test.go`
**Issue:** Removing `signal.NotifyContext` fails no test, and no row cancels during the window wait, the parent wait or a lock acquire. D12's criterion names a child ended by SIGTERM.

### [MEDIUM] Lowest-priority selection is unpinned

**Location:** `internal/kwinplace/recordings_test.go`
**Issue:** In both kscreen recordings DP-2, the priority 1 output, is also the first enabled one.

### [LOW] handTerminalWindow's main path is untested

**Location:** `internal/reedcli/handterminalwindow_integration_test.go`
**Issue:** Only the outside-a-worktree case calls it; the window verb's test stubs it and the child test calls `Spawn` directly.

### [LOW] An unusable parent desktop gives a permanent home

**Location:** `internal/kwinplace/planner.go` (`Anchor`), `internal/kwinplace/flow.go` (`placeLocked`)
**Issue:** A settled parent entry can hold no last desktop, as one settled by a failed resolution does. `Plan` then silently starts at the current desktop with `fallback` false, and the child records that as home. D8 keeps such a window homeless.

### [LOW] A minimized window loses its order key

**Location:** `internal/kwinplace/flow.go` (`reportUpdateFrom`)
**Issue:** The grid rebuild releases it, the planner does not count it, and the report orders it last, so the renumber moves its key.

### [LOW] One-row grid differs from the reference script

**Location:** `internal/kwinplace/apply.js` (`buildGrid`)
**Issue:** D6 says the grid is built as `set-tiling-grid.sh` builds it, and that script sets one-row column widths twice. The effect of one pass on three or more columns is unverified.

### [LOW] Planner boundaries without rows

**Location:** `internal/kwinplace/planner_test.go`
**Issue:** `next` from the last desktop reaches the `min` clamp that prevents an index panic, and an anchor naming no desktop reaches the current-desktop fallback; neither has a row.

### [LOW] Runner polling, arguments and cache key are unpinned

**Location:** `internal/kwinplace/runner_test.go`
**Issue:** A single journal read and a single `isScriptLoaded` read pass every row, a wrong journalctl unit passes, and so does a shared report reused for another output.

### [LOW] Lock-order tests do not force an overlap

**Location:** `internal/kwinplace/runner_test.go`, `internal/kwinplace/flow_test.go`
**Issue:** Instant fakes rarely interleave, so both tests usually pass with the lock removed.

### [LOW] tier3 still described as LLM-only

**Location:** `pattern/PATTERN-test-speed.md`, `contracts/specs/loom-plan-spec.md:346`
**Issue:** The table row now includes a desktop session; the prose below it and the plan spec do not.

### [NIT] Unreachable KWin reads as a leftover script

**Location:** `internal/kwinplace/runner.go` (`unload`)
**Issue:** `isScriptLoaded` errors are discarded, so the refusal says the script is still loaded when its state is unknown.

### [NIT] Small unpinned error paths

**Location:** `internal/desktophook/config_test.go`
**Issue:** The tile-height refusal, the report parse errors and the joined after-place error have no case.

## Rejected

- code-tests F10, `OpenWindow` passing `placed` to `Launcher.Open`: card 7 states the function has no branch and is pinned by `TestPreLaunch` and the launcher's command-line test.
- code-tests F10, `QueryScript`'s input names: a wrong name fails the real script only, which the tier3 query contract test drives.
- code-correctness F10 and code-affected-set F3 are one defect, kept once as F10.
- code-tests F11 and the handler's own reading of the contract test are one defect, kept once as F4.
