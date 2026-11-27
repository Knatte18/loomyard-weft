```yaml
lens: way-forward
outcome: reported
```

Lens: way-forward. I checked each refusal, stop and recovery step the cards add, and whether the next step it names can succeed from the state it leaves.

**Findings**

1. MEDIUM — Card 8 (`ReadPlaces`, `RecordOpen`, `Settle`, `RecordReport`): a corrupt places file never heals, and every child then waits out its parent.
   - Card 8 reads an undecodable file as empty, with a Warn.
   - Every write goes through `state.UpdateJSON`. On a decode failure that function "aborts before mutate runs, and no write happens" (`internal/state/state.go`, `UpdateJSON` doc).
   - So `RecordOpen`, `Settle` and `RecordReport` all fail for as long as the file stays corrupt, and no card names a step that rewrites or removes it.
   - The result: no window is ever recorded or settled, and every child whose parent sits in that hub waits the full 180 seconds before it falls back to the current desktop. D9 promises only "every window is then placed as new".
   - `TestPlacesFile` has no corrupt-file row.
   - Fix: on `errors.Is(err, state.ErrDecode)`, replace the file with the one new entry, or move the bad file aside, and add a `TestPlacesFile` row that pins it.

2. MEDIUM — Card 12 (`reedCLI.windowCmd`, `handTerminalWindow`): when the hand-off fails, `window_command` never runs.
   - With the pre-launch checks passing, `windowCmd` skips `placeWindow` and calls `handPlacement`, and "a hand-off error only warns".
   - When `kwinplace.Spawn` fails to start the child, nobody runs `desktop.window_command`: not the verb, and not a child that never started. Today that command always runs after a window opens.
   - The next step the card names, "the verb still succeeds", leaves the operator's script unrun. D12 only gives the command away to a child that has been handed the window.
   - The `HandOffFailureOnlyWarns` row does not assert that `placeWindow` runs.
   - Fix: on a hand-off error, fall back to `placeWindow` (wait, then `window_command`), and assert that in the row.

3. MEDIUM — Card 11 (`reedCLI.placeWindowCmd`): when a resolution step fails, the window is never recorded, so its children wait the full bound.
   - "A resolution that fails warns, still runs afterPlace … and exits." Examples are a refused `beam.window_placement` value, a failed `ParentOf`, or a failed `fabricengine.List`.
   - That exit happens before `Flow.Place`'s step 1, so the window gets no `RecordOpen` and no `Settle`.
   - For an orch window (the prime or a beam), every child window anchored on it reads a missing or stale entry. Each waits 180 seconds, then anchors on the current desktop and stays homeless.
   - The warning sits only in the hub log and names no way forward. A refused mode value should say to fix it with `lym config beam --set window_placement=<value>` or in the task's run_settings.
   - Fix: on a resolution failure, record and settle the entry (`RecordOpen` plus `Settle`) before `afterPlace`, and name the fix in the warning.

4. MEDIUM — Card 7 (`Runner.run` step 2): a leftover script that KWin never unloads blocks every later placement, with no way out named.
   - "A name still loaded after that wait fails the run before its load, and stays in the file."
   - If KWin keeps that script loaded, every later run on the machine, for every hub, fails the same way. No warning names how to clear it, such as `qdbus6 org.kde.KWin /Scripting unloadScript <name>` or deleting `kwinscript.loaded`.
   - The failure surfaces to the flow (card 10) only as a generic failed-run warning, so it repeats forever.
   - Fix: give this failure its own error that names the plugin name and the unload command, and assert the text in the matching `TestRunner_ScriptRuns` row.

5. LOW — Card 2 (`desktophook#LoadConfig`): a bad placement value takes down unrelated features, and the refusal names no fix.
   - `LoadConfig` now refuses a bad `window_placement` or a tile size below 1, but the refusal names no fix (the machine-layer `lym config desktop --layer machine --set <key>=<value>`).
   - The same load feeds `AfterWindow`, used by `lym reed window` and `lym ide`, and the status-bar buttons (`internal/hubgeom/buttons.go:44`).
   - So a typo in a placement key also stops the window command and the buttons, and the only message is the window-command warning.
   - Fix: add the set command to the refusal text and assert it in the `UnknownPlacementRefused` and `ZeroTileWidthRefused` rows.

6. LOW — Card 10 step 5 versus Card 7 `Runner.Query`: the "fresh Query" under `kwinplace.lock` has no `after` value.
   - `Query` returns a cached `kwinquery.json` report that ended after `after`.
   - If the under-lock query passes the zero time, or the window wait's last end, it can plan from a snapshot taken before another child's apply. Two placements could then fill the same zone, which D10's lock exists to prevent.
   - Fix: state that the under-lock query passes the time the lock was taken, or skips the cache, and add a `TestFlow_Place` row for it.

7. LOW — Card 10 step 6: the empty-target branch does not say it releases `kwinplace.lock`.
   - That branch "settles the entry with the plan's warning", but only the apply branch says "then release the lock".
   - The process exits right after, so nothing blocks, but the step is left implicit.
   - Fix: say both branches release the lock before `afterPlace`.

8. LOW — Cards 5 and 7 (`TilingOutput`, `QueryScript`): a tiling output KWin does not know shows up only as a timeout.
   - If `kscreen-doctor`'s name for the priority-1 output is not among KWin's `workspace.screens`, the query script has no output to tile and may never log its end marker.
   - Every placement then fails as `ErrScriptTimeout`, which names nothing about the output.
   - Fix: have the query script log a "no such output" line before its end marker, and have the flow warn naming the output.

**Tool calls**
- grep of `fabricengine.List`, `ReadOriginFor` and the `desktophook.LoadConfig` callers: I needed to know whether card 11's test PATH (only `sh`) breaks resolution, and who else a desktop load refusal reaches. The context lacked both. `List` reads through `dotgit` and spawns no git, so I raised no finding on that test.
