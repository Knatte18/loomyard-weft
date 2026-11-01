---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/ideengine/spawn.go:46
    origin: lens:code-correctness
    summary: A prime lym ide spawn passes the hub .code-workspace file as the window command's working directory, so the command never runs for the prime's VS Code window.
    fix: In internal/ideengine/spawn.go, make Spawn hand placeWindow a directory. For the prime, whose result.launchPath is the workspace file, pass filepath.Dir(result.launchPath), the hub directory; for a task slug keep the bare folder. Add a prime row to the hub integration test in internal/ideengine/spawn_hub_integration_test.go that asserts the marker file is written after a prime Spawn.
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/desktophook/window_test.go:25
    origin: lens:code-tests
    summary: No test pins that AfterWindow waits window_wait_ms before running the command, which is the Goal's requirement that the script sees the mapped window.
    fix: In internal/desktophook, make the wait injectable (an unexported sleep function or option AfterWindow passes through) and add a TestAfterWindow row with a nonzero window_wait_ms that asserts the injected sleep received that duration before the command ran. Keep the test free of real sleeps.
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/desktophook/window.go:19
    origin: lens:code-tests
    summary: The command timeout and output wait delay are package constants, so the hanging-command path that keeps the window open from blocking is untestable and untested.
    fix: In internal/desktophook/window.go, pass the timeout and wait delay into runWindowCommand as unexported parameters or fields, keeping 30s and 5s as the production values. Add a TestAfterWindow row with a hanging command (for example sleep 60) and a millisecond timeout that expects an error and a prompt return.
  - id: F4
    severity: MEDIUM
    class: scope
    location: internal/ideengine/spawn.go:97
    origin: lens:code-tests
    summary: OpenTask, the lym ide open opener, now runs the window command but no test calls it, and no test covers a failing CodeLauncher skipping the command.
    fix: In internal/ideengine/spawn_hub_integration_test.go, add a case that calls OpenTask with setDesktopWindowCommand and asserts exactly one marker run, and a case with a failing CodeLauncher that asserts the error is returned and the marker is absent. Add the failing-launcher case for Spawn in the same place.
  - id: F5
    severity: LOW
    class: scope
    location: internal/reedcli/window.go:34
    origin: lens:code-tests
    summary: placeTerminalWindow is replaced by a seam in every reedcli test, so its worktree resolution, ConfigDirs wiring and quiet nil for a directory in no worktree are never run.
    fix: Add a test in internal/reedcli for placeTerminalWindow on a directory outside any git worktree (returns nil, runs nothing) and, in a tier with a hub fixture, on a worktree directory (the configured command runs).
  - id: F6
    severity: LOW
    class: scope
    location: internal/desktophook/window_test.go:25
    origin: lens:code-tests
    summary: AfterWindow treats a whitespace-only window_command as unset, and no row pins it.
    fix: Add a TestAfterWindow row with window_command set to three spaces and window_wait_ms 0 that expects a nil error and, with a marker-writing variant of the check, no command run.
  - id: F7
    severity: LOW
    class: scope
    location: internal/configcli/configcli.go:484
    origin: lens:code-affected-set
    summary: The lym config help text was reworded from Keys the current template does not recognize are to Keys the template lacks are, a change the Goal does not ask for.
    fix: Revert the line in buildConfigLong in internal/configcli/configcli.go to its original wording, Keys the current template does not recognize are.
  - id: F8
    severity: NIT
    class: consistency
    location: internal/desktophook/window_test.go:25
    origin: lens:code-tests
    summary: The UnsetCommandRunsNothing row checks for a marker file, but its machine config never writes one, so the marker check cannot fail.
    fix: Drop the marker check for that row and keep its nil-error assertion, or give the row a window_command that would write the marker if it ran, such as the blank-command row of F6.
---

### [BLOCKING] Prime ide spawn runs the window command in a file path

**Location:** internal/ideengine/spawn.go:46
**Issue:** `seed` sets `launchPath` to the hub workspace file for the prime (`internal/ideengine/spawn.go:90`), and `Spawn` hands it to `placeWindow`, which passes it to `desktophook.AfterWindow` as `dir`. `runWindowCommand` sets it as `cmd.Dir` (`internal/desktophook/window.go:55`), and a file there fails the start with ENOTDIR. The failure is only warned, so the prime's VS Code window, the common `lym ide spawn` case, never gets the command. The Goal names `lym ide` as an opener; the task slug path works and is the only one the integration test covers.

### [MEDIUM] The configured wait is unpinned

**Location:** internal/desktophook/window_test.go:25
**Issue:** Every `TestAfterWindow` row sets `window_wait_ms: 0`, so removing the `time.Sleep` at `internal/desktophook/window.go:35` fails no test. The Goal requires the command to run once the window is mapped or after a short wait.

### [MEDIUM] The timeout path is untestable and untested

**Location:** internal/desktophook/window.go:19
**Issue:** `commandTimeout` and `commandWaitDelay` are constants, so no test can exercise a hanging command without a 30 second run. The focus directive asks that a hanging command never block the window open; the code bounds it, but nothing pins the bound, and the testing rules require an injectable timeout.

### [MEDIUM] OpenTask and failed launches have no test

**Location:** internal/ideengine/spawn.go:97
**Issue:** `checkTaskSlugOpensBareFolder` covers `Spawn` and `Seed` only. `OpenTask` (`lym ide open`) runs the window command with no test, and no test shows a failing `CodeLauncher` skipping the command.

### [LOW] placeTerminalWindow never runs under test

**Location:** internal/reedcli/window.go:34
**Issue:** The `placeWindow` seam replaces it in all of `window_test.go`, so its `ResolveWorktree` call, `ConfigDirs` wiring and the quiet nil for a directory in no worktree have no coverage. `lymcwd.ResolveWorktree` walks up to the worktree root through `gitWorktreeRoot`, so a subdirectory resolves; that part is not a defect.

### [LOW] Blank window_command is unpinned

**Location:** internal/desktophook/window_test.go:25
**Issue:** The `strings.TrimSpace` check at `internal/desktophook/window.go:31` has no row; the unset row uses an absent key.

### [LOW] Unrelated help-text rewording

**Location:** internal/configcli/configcli.go:484
**Issue:** The rewording has nothing to do with the window command, and the Goal does not license it.

### [NIT] Vacuous marker check

**Location:** internal/desktophook/window_test.go:25
**Issue:** `UnsetCommandRunsNothing` sets no command, so no marker could ever be written and the check proves nothing beyond the nil error.

## Rejected

- Lens code-correctness F2, `placeTerminalWindow` failing for a subdirectory of a worktree: `lymcwd.ResolveWorktree` calls `resolveCore`, which finds the root through `gitWorktreeRoot`, a walk up from the told path (`internal/lymcwd/lymcwd.go:133`, `:170`), so a subdirectory resolves.
- Lens code-correctness, unbounded `window_wait_ms` blocking the verb: a machine-local misconfiguration, not a defect in the diff.
- Lens code-tests item 9, no `AfterWindow` row for a failing config load: the wrapping is one `fmt.Errorf`, and the `LoadConfig` error rows already pin the failure itself.
- Focus question on splitting the command string: the Goal calls the key a command, and `desktophook` documents and runs it through the platform shell on purpose, so `~` and arguments work as the Goal's own example needs; no unintended shell surprise found.
- Focus question on KDE-specific code: a search of the diff finds no KDE, KWin or Plasma naming outside the commit and docs text, so it is not a defect.
- Focus question on `Pattern-Departure` trailers: neither commit carries one, and no declared PATTERN change is listed in the directive, so nothing to judge.
