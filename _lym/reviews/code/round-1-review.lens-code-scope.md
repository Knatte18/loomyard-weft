```yaml
lens: code-scope
outcome: reported
```

Task: trace each change in the diff to the decision or goal point that asks for it (code-scope lens).

**Decisions (board entry `## Goal` points, plus the `Added after Discussion` entries):**

- **G1:** `lym machine start [<shortname>]` starts the prime and owned beams of one hub or every live hub, through `lym beam start` in the hub's prime, as a child named through `selfexec.LymBinary`.
  - Built by `internal/machinecli/start.go` (`runStart`, `startHub`, `runBeamStarts`, `runBeamStart`), `cli.go` (the `start` cobra verb and its seams) and `hubs.go` (`liveHubEntries`, `lookupHub`).
- **G2:** it skips a prime another machine owns, or one with no recorded owner, says so, and never goes through `lym s`.
  - Built by `start.go` `planStart`.
  - Whether a beam started by slug under an unowned prime still claims that prime is a correctness question for the code-correctness lens, so I leave it out here.
- **G3:** it opens the hub's workspace file and one terminal per started worktree running `lym reed attach`, or prints each attach command where no launcher resolves.
  - Built by `start.go` `openWindows`, with the `terminal` and `openWorkspace` seams in `cli.go`.
- **G4:** it prints a standing hold with its machine and subjects, and the `lym adopt <subjects>` re-run for this machine's own hold.
  - Built by `start.go` `holdOf`.
- **G5:** `termwindow` gains native Windows (`wt.exe`).
  - Built by `internal/termwindow/termwindow.go` (`resolve`'s windows branch, `nativeWindowsTerminalArguments`, `escapeWindowsTerminal`) and its test rows.
- **G6:** `internal/desktoplauncher` writes `.desktop` files on Linux and `.cmd` files on the Windows desktop, through `wsl.exe` under WSL.
  - Built by `desktoplauncher.go`, `locate.go`, `locate_other.go`, `locate_windows.go`, `doc.go` and the tests.
- **G7:** the launcher is added at the dobby start and removed by every catch-up, the dobby start's included, that finds this machine owns nothing in the hub, inside `internal/dobbyengine`. The verb also adds and removes it. Nothing of it lives in `internal/beamcli`.
  - Built by `internal/dobbyengine/launcher.go` (`addLauncher`, `removeLauncherIfOwnsNothing`, `OwnsAnything`, `Launcher`), `catchup.go` (the `Launcher` field and the removal call), `loop.go` (`d.addLauncher()` in `run`) and `internal/battencli/catchup.go` (`HubCatchUp` fills `Launcher`).
  - The verb side is `start.go` `keepLauncher` and `removeGoneLaunchers`.
  - `internal/beamcli` is untouched.
- **G8 (Exports):** `desktoplauncher`'s add and remove functions and `machinecli`'s start verb.
  - Built by `Desktop.Add` and `Desktop.Remove` (plus `Shortnames`, which `removeGoneLaunchers` needs) and `newCommand`'s `start`.
- **Added after Discussion entries:**
  - "Hub stop writes no per-run pause or cancel intent" (D9) and "Only the printed parent_beam names the prime root" (D8) belong to other parts of the Sketch. This part builds neither and none is asked of it.
  - The two arbiter accept entries carry no code change.

**Changes in the diff, mapped:**

- `machinecli/start.go`, `cli.go` (start verb and seams) → G1–G4 and G7's verb side.
- `machinecli/hubs.go` (`recordDir` split, `liveHubEntries`, `lookupHub`) → G1; the `liveHubs` refactor is forced by it.
- `machinecli/doc.go` and the `docs/overview.md` rows → forced docs for G1–G7.
- `termwindow/termwindow.go` and its test → G5.
  - The `arguments(dir, title, lymPath)` signature change, which moves the Konsole and WSL rows onto `attachLine` inside each builder, is forced by G5: the commit message says native Windows runs the attach with no shell line.
- `desktoplauncher/*` → G6 and G8.
  - `FormWindows` (a native Windows `.cmd`) traces to G6's ".cmd files on the Windows desktop".
- `dobbyengine/launcher.go`, `catchup.go`, `loop.go`, `doc.go` → G7.
- `dobbyengine/seam_enforcement_test.go` allowlist (`desktoplauncher`, `machinename`, `selfexec`) → forced by G7.
- `battencli/catchup.go` `HubCatchUp`'s `Launcher` → G7, wiring every catch-up site (the dobby, a hand pass, adopt).
- `testkit/tmuxkit` `XDG_DESKTOP_DIR` and `TestSetEnv` → forced by G6 and G7, so no test writes the operator's desktop (PATTERN-test-isolation).
- The verb's board pull before it reads owners (`startHub`'s `s.d.pull`) is forced by PATTERN-owner-machine's catch-up-after-absence rule, not unrequested scope.

**Findings:** none. Every change traces to a goal point or is forced by one, and every goal point has a change that builds it.

Tool calls: none.
