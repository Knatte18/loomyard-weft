```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the diff, running each changed function and branch against a concrete case.

**Functions and branches run**

| Function / branch | Case run | Result |
|---|---|---|
| `Flow.Place` | RecordOpen fails; ctx cancelled; older child ends after a newer one | F1 |
| `placeWindow`, `takeLock`, `acquireWithin` | budget spent, cancel, timer racing the result | correct |
| `awaitWindow` | first probe false, query error, cancel during sleep | correct |
| `placeLocked` | off mode, parent unsettled, parent settled with no desktop, Apply fails | F4 |
| `awaitParent` | deadline passed, unchanged or changed file | correct |
| `reportUpdateFrom` | zero plan, sticky window, minimized window on the target | F5 |
| `updatePlaces` | gone home, stale worktree list, hand move, renumber | F2, F3, F5 |
| `renumber`, `rehome`, `RecordOpen`, `Settle`, `writePlaces` heal, `ReadPlaces` | empty or undecodable file, duplicate match | correct apart from F1 |
| `Plan`, `targetOrder`, `Capacity`, `GridFor`, `Anchor`, `CurrentOrder` | full desktops, past the last desktop, equal keys | correct apart from F2, F4 |
| `Runner.Query`, `runScript`, `runLocked`, `unload`, `readReport`, `call` | load -1, timeout, cancel, leftover, KWin unreachable | F9 |
| `ReportLines`, `ParseQueryReport`, `ParseApplyReport`, `TilingOutput`, `fillScript` | no-output, unknown kind, no enabled output | correct |
| `query.js` | missing output, one desktop | correct |
| `apply.js` `buildGrid` and report | 1x1, 1xN, 2xN | F7 |
| `PreLaunch`, `OpenWindow`, `Launcher.Open` | each check failing, start error | correct |
| `Spawn` | start or registration failure, prime | correct |
| `handTerminalWindow`, `windowCmd` | relative dir, plain checkout, hand-off error | F6, F8 |
| `placeHandedWindow`, `placementRequest`, `recordUnplacedWindow` | unresolved worktree, refused mode, cancel | correct apart from F3 |
| `openWindows` | hand-off error, unplaced window | correct |
| `AfterPlacement`, `runWindowCommand` | cancel, timeout, caller group killed | F10 |
| desktop `LoadConfig`, dobby lookup, `beamsetting` | `sideways`, 0, `""` | correct |

**Findings**

**F1. MEDIUM, design.** `internal/kwinplace/places.go` (`Settle`, `updatePlaces`).
- An older child that finishes after a newer window's `RecordOpen` overwrites the newer window's entry. Both functions write `SettledBy` = the old pid without comparing it to `entry.PID`.
- If the newer child has already settled the entry, it reads unsettled again, so children of that parent wait the full 180 s.
- `LastDesktop`, and possibly `Home`, also take the old window's values.
- Concrete case: window A waits for its parent, the worktree's window is reopened as B, B finishes first, then A finishes.
- D9 says an entry is settled by the current window's child.
- Evidence: `places.go` `Settle` (`entry.SettledBy, entry.RecordedAt = pid, now`) and `updatePlaces` (`placed.SettledBy ... = update.PID`).

**F2. MEDIUM, design.** `internal/kwinplace/places.go` (`updatePlaces`).
- A window whose home desktop no longer exists never gets a new home. `rehome` runs only when `placed.Home == ""`.
- `Plan` treats the gone home as no home (`fromHome` false, `Overflowed` false), but the entry keeps the dead id.
- Every later reopen is therefore placed by mode, waits for the parent again, and keeps no order key.
- D8 calls this "a window without one", and D9 says where it landed becomes its home when it had none.
- Evidence: `places.go` `if placed.Home == "" && !update.Fallback && !update.Overflowed`; `planner.go` `fromHome := start >= 0`.

**F3. MEDIUM, design.** `internal/reedcli/placewindowchild.go` (`placementRequest`) with `places.go` (`updatePlaces`).
- The worktree list is read when the child starts, but applied to the places file after up to 10 + 180 + 120 s of waiting.
- A pair created and opened during that time has its entry deleted by `DeleteFunc` as a worktree "the hub no longer has".
- Its home and order key are lost, and a parent entry deleted this way reads as missing.
- Its own later `Settle` or `RecordReport` recreates the entry with no `Title`. `Window.Matches` then accepts any caption for that pid.
- This is the ordinary batten case of run windows opening while other children wait.
- Evidence: `placewindowchild.go` `fabricengine.List` into `Request.Worktrees`; `places.go` `maps.DeleteFunc(... !slices.Contains(update.Worktrees, worktree))`.

**F4. LOW, design.** `internal/kwinplace/flow.go` (`placeLocked`), `planner.go` (`Anchor`, `Plan`).
- A settled parent can have `LastDesktop` `""` and no live window, for example when its window never appeared or sat on all desktops. A removed `LastDesktop` behaves the same.
- `Anchor` returns `known = true` with an unusable desktop, and `Plan` silently uses the current desktop.
- `fallback` stays false, so the child records that desktop (or the one after, under `next`) as its permanent home.
- D8 keeps a window anchored on the current desktop for want of a parent's place homeless.
- Evidence: `planner.go` `return entry.LastDesktop, true` and `start = max(0, indexOf(in.Snapshot.CurrentDesktop))`.

**F5. LOW, design.** `internal/kwinplace/apply.js` (`buildGrid`) with `flow.go` (`reportUpdateFrom`).
- `buildGrid` releases every tiled window, but a minimized window is not counted, so it is not snapped back.
- In the report it is untiled, and `CurrentOrder` normally puts it last (or in whatever zone its old frame corner matches).
- `updatePlaces` then sees the order disagree and renumbers the minimized window's key, although the operator moved nothing.
- Case: homed keys 0 (minimized), 1 and 2; reopening key 2 rewrites the minimized window to 2.
- Evidence: `planner.go` `counted` excludes `Minimized`; `flow.go` builds the order over all `onDesktop` windows.

**F6. MEDIUM, design.** `internal/reedcli/window.go` (`handTerminalWindow`).
- A relative `<worktree-dir>` is passed straight to `lymcwd.ResolveWorktree`, which keeps it relative (`EvalSymlinks`, then `FindRoot`).
- For `lym reed window ../task`, `HubPath` is `..`. `Spawn` sets `cmd.Dir` to `..` and also passes `--hub ..`, so the child resolves the hub's parent.
- The child then writes `_bolt/.lym/window-places` there and places nothing. The window command, which ran before this change, no longer does.
- For `.`, the worktree name becomes `.`.
- Evidence: `window.go` `dir := args[0]` with no `Abs`; `lymcwd.go:202-212`; `child.go` `cmd.Dir = hubPath`.

**F7. LOW, design.** `internal/kwinplace/apply.js` (`buildGrid`).
- For one-row grids the columns' `relativeGeometry` is set in one pass only.
- The operator's `~/.local/bin/set-tiling-grid.sh`, which D6 says the grid is built like, sets it a second time when `rows == 1`. Its own comment says the split tree starts skewed (0.5, 0.25, 0.15, 0).
- A single left-to-right pass is therefore untested for 3 to 5 columns; the one-row recording has only 2. I did not drive it, so whether the widths actually come out wrong is unverified.
- Evidence: `apply.js` `if (columns.length > 1)` followed by `if (rows > 1)` with no else; the script's `else { col.relativeGeometry = ... }`.

**F8. MEDIUM, design.** `internal/reedcli/window.go` (`handTerminalWindow`).
- A directory in a git repository that is not in any hub resolves successfully, with `HubPath` = its parent folder.
- The child is spawned and creates `<parent>/_bolt/.lym/window-places` and log files under `<parent>/_bolt`, for example in `~/Code`.
- Card 13 says a directory with no hub hands nothing, and `PATTERN-lym-files` says a plain checkout gets no `.lym` that lym does not own.
- Evidence: `window.go` treats only a `ResolveWorktree` error as "no hub"; `lymcwd.go` `buildLocation` uses `filepath.Dir(workTreeRoot)` unchecked; `placewindowchild.go` `SetDurableSinkDir` and `RecordOpen`.

**F9. NIT, design.** `internal/kwinplace/runner.go` (`unload`, `runLocked`).
- When KWin or `qdbus6` is unreachable and a leftover name is recorded, every `isScriptLoaded` call errors.
- The result is reported as `LeftoverScriptError`, "is still loaded", with a by-hand unload as the way forward. The real error is only a warning from the unload call; the `isScriptLoaded` errors are dropped.
- Evidence: `runner.go` `if err == nil && strings.TrimSpace(...) == "false"`, with no handling of `err`.

**F10. LOW, design.** `internal/desktophook/window.go` (`runWindowCommand`).
- The command now runs in its own process group for `AfterWindow` callers too.
- A Ctrl-C, or a group kill of `lym reed window` or `lym ide` by their caller (such as a bounded command row), no longer reaches the command. It survives, and nothing enforces the 30 s bound once the parent is dead.
- Card 4 says `AfterWindow` keeps its behaviour.
- Evidence: `window.go` `proc.ConfigureGroupTerminate` on the shared runner; `procgroup_linux.go` `Setpgid: true`.

**Tool calls**
1. One Bash read of `gitWorktreeRoot` and `buildLocation` in `lymcwd.go`, the head of `dotgit.FindRoot`, and `~/.local/bin/set-tiling-grid.sh`. The context lacked whether resolution makes a path absolute, and the reference script D6 names.
