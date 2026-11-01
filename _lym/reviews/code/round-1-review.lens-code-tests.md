```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I name the test that pins it, or "none".

**Per-behaviour list**

1. Template defaults (`window_command` empty, `window_wait_ms` 1500). Pinned by `TestLoadConfig/TemplateDefaults` (`internal/desktophook/config_test.go`).
2. A machine override sets both keys. Pinned by `TestLoadConfig/MachineOverrideSetsBothKeys`.
3. A negative `window_wait_ms` is refused with `ErrInvalid`, naming the key. Pinned by `TestLoadConfig/NegativeWaitRefused`.
4. A local override of a machine key is refused. Pinned by `TestLoadConfig/LocalOverrideRefused`.
5. `LoadConfig` falls back to the template when the hub base file is absent. Pinned: the `newDirsWithMachineConfig` fixture holds no hub file, so `TemplateDefaults` fails if the fallback breaks.
6. `AfterWindow` with an unset command runs nothing. Pinned by `TestAfterWindow/UnsetCommandRunsNothing`. The row's marker check is vacuous, because that machine config never writes a marker. It would still fail if the empty command reached the shell and returned an error.
7. `AfterWindow` runs the command in the window's directory. Pinned by `TestAfterWindow/SetCommandRunsInWindowDir`.
8. A failing command's error carries its output. Pinned by `TestAfterWindow/FailingCommandReportsOutput`.
9. `AfterWindow` returns an error when the config fails to load, wrapped with "load desktop config". None. `TestAfterWindow` has no row with an invalid machine config, and the `LoadConfig` error rows go through `LoadConfig` directly. Minor, because the wrapping is trivial.
10. A blank command (whitespace only) is treated as unset, via `strings.TrimSpace`. None. The unset row uses an absent key, not `"   "`. Minor.
11. `window_wait_ms` is slept before the command. None. Every `AfterWindow` row sets `window_wait_ms: 0`, so removing the `time.Sleep` fails no test. This is the Goal's "only once the new window is mapped, or after a short wait" requirement.
12. `commandTimeout` and `commandWaitDelay` bound a hanging command. None. No test runs a hanging command. This is the focus directive's "hanging command never blocks" point. The constants are not injectable, so a test would take 30 seconds. The design is untestable as written (`commandTimeout` is a package constant, so a test cannot shorten it); this belongs to the testing rule "a production interval, timeout or retry delay is injectable".
13. `reed window` runs the placement hook after a successful open and passes the told `dir`. Pinned by the `wantPlaced` assertion in `TestWindowCmd_OpensATerminalAttachedToTheToldWorktree` (`internal/reedcli/window_test.go`), for the `DefaultTitleIsTheBaseName` row and the other success rows.
14. `reed window` does not run the hook when the launcher fails or the directory is missing. Pinned by the `wantPlaced` empty case in the `MissingDirectoryRunsNoLauncher` and `LauncherErrorIsCarried` rows.
15. A hook failure in `reed window` only warns, with exit code 0 and an ok envelope. Pinned by `WindowCommandFailureOnlyWarns`.
16. `placeTerminalWindow` resolves the worktree from `dir`, uses `ConfigDirs`, and returns nil quietly when `dir` is in no worktree. None. The `placeWindow` seam replaces it in every `reedcli` test, so the production function is never run. It is covered at a distance by `TestAfterWindow`, but not its resolve step or its no-worktree branch.
17. `ideengine.Spawn` runs the hook after a successful launch, and `Seed` does not. Pinned by `checkTaskSlugOpensBareFolder` in `internal/ideengine/spawn_hub_integration_test.go` (marker absent after `Seed`, marker present after `Spawn`). This is for the task-slug path only. I did not verify that the prime path calls this helper, so it may have no assertion that the hook runs after a prime spawn's launch.
18. `ideengine.OpenTask` runs the hook after a successful launch. None found in the diff. The hub integration test change covers `Spawn` and `Seed`, with no `OpenTask` call and no marker assertion.
19. The hook is not run, and the error is returned, when `CodeLauncher` fails in `Spawn` or `OpenTask`. None. The diff adds no row with a failing `CodeLauncher`.
20. A hook failure in `ideengine` only warns (`placeWindow`). None. The integration test only runs a succeeding command.
21. `configreg` registers the `desktop` module as hub-wide with the machine keys. Pinned by `TestModules_SeedOnlyAndHubWideFlags` and `TestModules_EveryTemplateKeyClassified` (`wantMachineKeys` row for `desktop`).
22. `desktophook` is in the degrading callers set. Pinned by the `configStrictnessDegradingSet` entry in `cmd/lym/configstrictness_test.go`, assuming that test checks the set against actual `LoadOrTemplate` callers.
23. Doc and comment edits in `configcli`, `configengine/doc.go`, `layers.go`, `PATTERN-config-layers.md` and `docs/overview.md`. No test is needed, since this is static prose.

**Findings**

- **F-tests-1, MEDIUM, design.** Nothing pins that `AfterWindow` waits `window_wait_ms` before running the command (item 11). Location: `internal/desktophook/window_test.go:25-27`; evidence: all three rows use `window_wait_ms: 0`, and `window.go:35` holds the sleep. Fix: add a way to observe the wait. For example, make the sleep injectable, or add a row with a small wait (such as 50 ms) that the command records with a timestamp or that the test measures against a lower bound. It must stay deterministic.
- **F-tests-2, MEDIUM, design.** The timeout and hang path is untested (item 12). `commandTimeout` and `commandWaitDelay` are non-injectable package constants (`window.go:19-22`), which also fails the testing rule that timeouts be injectable. Fix: pass the timeout through an unexported option or a parameter, and add a row with a hanging command and a short timeout that expects an error and a prompt return.
- **F-tests-3, MEDIUM, scope.** `OpenTask` runs the hook with no test (item 18), and the failing-`CodeLauncher` paths for `Spawn` and `OpenTask` have none (item 19). `ide open` is a Goal opener. Location: `internal/ideengine/spawn.go:97-107`. Fix: add a hub-integration row calling `OpenTask` with the marker command and asserting one run, and a row with a failing `CodeLauncher` asserting the marker is absent and the error is returned.
- **F-tests-4, LOW, scope.** `placeTerminalWindow` is untested (item 16): the resolve step, the `ConfigDirs` wiring, and the quiet nil for a directory in no worktree. Location: `internal/reedcli/window.go:34-41`. Fix: add a test in `reedcli`, or in a tier that has a hub fixture, covering a directory in no worktree (returns nil) and a worktree directory (command runs).
- **F-tests-5, LOW, scope.** `AfterWindow` has no row for a failing config load or a blank command (items 9 and 10). Location: `internal/desktophook/window_test.go:25-27`. Fix: add rows for `window_command: "   "` (expects nil and no marker) and `window_wait_ms: -1` with a command (expects an error and no marker).
- **F-tests-6, NIT, consistency.** The `UnsetCommandRunsNothing` row's marker check cannot fail, because that machine config writes no marker. Location: `internal/desktophook/window_test.go:25`. Fix: use a command that would write the marker if run, such as `window_command: "   "` (blank), or accept the row for its nil-error assertion.

**Tool calls:** none. I made no tool calls; the inherited context already held every diff and test file I used. Item 17's prime-path coverage is unverified because the full `spawn_hub_integration_test.go` is not in my context.
