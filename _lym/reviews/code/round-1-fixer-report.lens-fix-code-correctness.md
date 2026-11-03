```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix's code changes through the correctness lens (run each changed branch, check callers, check each added test fails with its defect in place).

No findings.

Walkthroughs:
- **`envconfig.LoadConfig`:** `PATH`, `Path`, `lym_parent` and `LYM_STRAND_NAME` are refused. `COMMONLIB_REPO` and `MODE` still load.
  - The error still carries `"vars.&lt;name&gt;"`, so the old rows and the `BAD-NAME` assertions in `driftengine` and `gatecli` still match.
  - The grep found no other use of `vars.PATH` or a lowercase `lym_` variable in an env config.
- **`run.go` reorder:** `withBuildServerOff` now runs after the module's entries. It deletes any module entry for a switch name and appends lym's value last. `enclosingGateEnv` handling is untouched.
- **F1 test (shared build):** the build gets `BUILD_MARK=1` from its own env, `GOFLAGS=-from-env-module` over the inherited `-inherited`, and `MSBUILDDISABLENODEREUSE=1` over the module's `0`.
  - Dropping the module from the build path leaves `GOFLAGS` as `-inherited`, so it fails.
  - Undoing the reorder gives `0`, so it fails.
  - The tier record shows the module's `BUILD_MARK`, because the build's env does not reach the tier.
- **F2 test (hub layers):** the hub-file-only run needs the hub file loaded, and an anchor-only load leaves `BUILD_MARK` empty. The second run needs the machine file to win. Either regression fails the test.
- **F3 tests (Drift wiring):**
  - `driftcli` expects three env lines: main capture child, compare, side capture. The failed `capture-child` at the worktree root runs no command, so it adds none.
  - Reverting any one of `takeRun`, `runCaptureChild` or `driftDeps` to `cfg.Step` leaves `DRIFT_MODULE` empty on that line and fails.
  - The `loomcli` test expects two lines, from capture and compare through `stepOf`.
  - The `StepWithHubEnv` unit row fails if the load error is swallowed. The `vars.BAD-NAME` text matches the message.
- **F4 test (launch ordering):** the expected line is built from `sh.Chain(exports, cmd)`.
  - Swapping the `sh.Chain` arguments or dropping the line at `spawn.go:119` changes the script and fails the exact comparison.
  - A nil `paneEnv` exports nothing, so the other rows are unchanged.
  - The test only reaches the host dialect, because `launchStrandLocked` fixes `shell.ForGOOS()`. That is a limit of the proposal, not a defect in the test.
- **F5 test (broken env config):** `BAD-NAME` makes `LoadConfig` fail, so the gate exits 1 with the refusal text and runs no tier. Removing the refusal fails it.
- **F6 test (hubgeom `PaneEnv`):** a nil `PaneEnv` fails the `Fatal`. A missing machine override or hub file fails the `DeepEqual`.

Further calls:
1. Grepped the tree for `vars.PATH`, `vars.&lt;lowercase&gt;=`, `never with LYM_` and `lym_&lt;name&gt;:` to find callers or fixtures the PATH and case-insensitive refusal could break. My context lacked a repo-wide search for these.
