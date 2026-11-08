---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/reedengine/buttonclick_test.go:178
    summary: 'no test shows the button prompt re-entry refuses a test binary handed in as the lym executable'
    fix: 'In the table of the test that builds buttonDeps at buttonclick_test.go:178, add a row whose executablePath is os.Executable instead of the stand-in exe. Assert that runButtonVia raises no prompt (no process run, no display-message prompt call) and logs "could not resolve this binary". Reuse the existing exeErr row expectations.'
  - id: F2
    severity: LOW
    class: consistency
    location: internal/loomcli/cli_test.go:355
    summary: 'the loopExecutable and reedcli window tests inject their own lookup, so the production default that wraps selfexec.LymBinary is never exercised'
    fix: 'Add a loopExecutable row with a zero loomCLI (lymBinary nil) that asserts the result is empty under go test, because the default lookup refuses the test binary. Add a window verb row in internal/reedcli/window_test.go with lymBinary nil that asserts the error starting with "window: resolve the running lym" and that the launcher is not called.'
---

### [LOW] Button prompt re-entry has no test-binary row

**Location:** internal/reedengine/buttonclick_test.go:178

**Issue:** `runButtonVia` resolves its binary through `selfexec.ResolveLymBinary(deps.executablePath)` (`internal/reedengine/buttonclick.go:156`), which is correct and makes the `internal/reedengine/doc.go` pin sentence true.
Every `buttonDeps` in the tests returns a stand-in path or a plain error.
No test hands the test binary in, so a revert to a bare `deps.executablePath()` would pass.
The pane, window, switch, follow, button and Slack pins all have such rows in `panebin_test.go` and `bindings_test.go`; this line does not.

### [LOW] Default lookups of loopExecutable and the window verb are untested

**Location:** internal/loomcli/cli_test.go:355

**Issue:** The `loopExecutable` test sets `lymBinary` to a lookup that fails with its own error.
The window test sets `lymBinary` to a stand-in path.
Neither reaches the `nil` default (`selfexec.LymBinary`), which is the line that keeps the test binary out.
Swapping the default for `os.Executable` would pass both tests.
The fronttask `startAction` test has the same shape, but its default is passed at one call site in `HubSettleDeps`, so no cheap row covers it and I raise no finding on it.

## Settled focus questions

- **Spawn and pin sites.**
  `creelcli/spawn.go`, `gatecli/detachfront.go`, `reedcli/remove.go`, `reedcli/window.go`, `boardengine/spawn.go`, `ideengine/spawn.go` and `battencli/wire.go` (the `loom start` spawn at line 1460) resolve through `selfexec.LymBinary` or a lookup defaulting to it.
  The remaining direct `os.Executable` calls are: `loomcli/drivewiring.go` and `battencli/wire.go:1039`, which `PATTERN-child-processes` names as resolve-once exemptions;
  `hubgeom` (dobby and Slack geometry), whose spawners refuse under go test (`dobbyengine/start.go:273`, `slackengine/start.go:95`);
  and the reads of the binary itself in `fabricengine.BinaryLabel`, `statuspane.CurrentBuild`, `adoptcli.runningLymBuild`, `tendercli/deps.go`, and `reedcli/watchdog.go`.
  None of them starts a test binary. Not a defect.
- **Button prompt and doc.**
  `buttonclick.go` resolves through `ResolveLymBinary` over the injectable lookup.
  The `doc.go` sentence is true for every pin, and it names `SpawnWatchdog` as the one direct `LymBinary` call. Not a defect; only the missing test is F1.
- **Process cap test.**
  `TestRunTierProcessCap` reaches a tripped cap through `runTier` via the `processCapFor` seam.
  It asserts exit 1 for a command that exited 0, `processCapped`, `failedOnShortage` returning empty over a log holding the shortage marker, the cap line in both stderr and `causes`, and the absent-run-marker case starting no cap. Not a defect.
- **tmuxkit run directory.**
  `Main` calls `lymtemp.Remove` before returning 2, after `HermeticGitEnv` made the directory.
  `reexecguard_integration_test.go` runs a real child with a private `TMPDIR` and asserts the run directory is empty. Every TestMain calls `tmuxkit.Main` (`allowedNoTmuxMain` is empty), so removing the gitkit and reedengine copies leaves no gap. Not a defect.
- **TestBinaryExecutable rows.**
  The reedengine rows now cover the Slack lines, follow, button and the switch bindings, plus the pane launch line and window command, and the real-tmux boot test checks all three option lines. The loomcli and fronttask tests show a lookup failure, not a test binary; see F2.

## Rejected

- The `PATTERN.md` entry line does not list the exemptions that `pattern/PATTERN-child-processes.md` names. The entry sits at its length cap and the background carries the detail, so this is not a finding.
- `internal/loomcli/start.go:598` still says "an os.Executable()" in a comment about the watchdog spawn. The wording is a past-tense description of the spawn's cost, not a rule, so it is not worth a finding.
