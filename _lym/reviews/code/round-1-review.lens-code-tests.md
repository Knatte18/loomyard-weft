```yaml
lens: code-tests
outcome: reported
```

**Per behaviour: the test that pins it, or none**

1. Template default is `auto`: `TestLoadConfig_Template` (prepare_test.go ~736) pins it.
2. `LoadConfig` accepts `auto` and refuses other values: the invalid-value row at prepare_test.go:568-570 pins the refusal with `read-only`. The updated message still contains `sandbox_mode "read-only"`, so the row still passes. No row loads an explicit `sandbox_mode: auto` through `LoadConfig`, but `TestPrepare_SandboxMode` does so through `Prepare`, so that is covered.
3. Machine layer may set the key and the local layer may not: `TestLoadConfig_SandboxModeLayers` (prepare_test.go:745) pins it. The diff changes only a comment there.
4. `runSandboxProbe` command shape, `CODEX_HOME`, cwd and exit 0 versus non-zero with output as the cause: `TestRunSandboxProbe` rows `SandboxStarts` and `SandboxCannotStart` pin them.
5. `runSandboxProbe` 30 s timeout reading as "cannot start": none. The timeout is a const and not injectable. Cheap to leave, so I raise it only as LOW.
6. `runSandboxProbe` failure with empty output, such as a missing binary (the `fmt.Errorf("%s sandbox: %w")` branch): none. The fake always prints the cause on non-zero exit.
7. `Prepare` with `auto` and a sandbox that starts, giving `workspace-write`: `TestPrepare_SandboxMode/AutoSandboxStarts`.
8. `Prepare` with `auto` and a sandbox that cannot start, giving `danger-full-access` plus a warning naming the cause: `AutoSandboxCannotStart`.
9. `Prepare` with explicit `workspace-write`, probe arguments (`codex`, lym home, anchor), start and refusal: `WorkspaceWriteSandboxStarts` and `WorkspaceWriteSandboxCannotStart`. The refusal row checks the cause, "way forward", the `--set sandbox_mode=auto` command and that no `prompt.md` was written.
10. `Prepare` with `danger-full-access` runs no probe: `FullAccess`, which asserts an empty `probed`.
11. `Prepare` default launch line is `workspace-write` via the fixture probe: `TestPrepare_Launch`.
12. `OpenLaunch` with the default and a probe that starts: the default `TestOpenLaunch` row, which uses the fixture probe.
13. `OpenLaunch` with `auto` and a probe that fails, giving `danger-full-access`: `AutoSandboxCannotStart`.
14. `OpenLaunch` with `danger-full-access` set: the existing row at open_test.go:50. It never asserts that no probe ran.
15. `OpenLaunch` with explicit `workspace-write` and a failing probe refusing the launch: none. `OpenLaunch` has its own `launchSandboxMode` call and error return (codexengine.go:190-192), and no `TestOpenLaunch` row sets `workspace-write` with a `probeErr`.
16. `OpenLaunch` calls the probe with the codex binary, lym home and anchor: none. The rows do not record the probe's arguments. The shared `launchSandboxMode` is pinned through `Prepare`, so this is LOW at most.

**Findings**

- F-tests-1, MEDIUM, design. Location: `internal/shuttleengine/codexengine/open_test.go` (`TestOpenLaunch` table, near line 54). The explicit `workspace-write` refusal on the `OpenLaunch` path (codexengine.go:190-192) has no row, so removing that error return would not fail any test. Evidence: the only probe-failure row in `TestOpenLaunch` is `AutoSandboxCannotStart`, which uses the default `auto`. Fix: add a row that writes `sandbox_mode: workspace-write` with a `probeErr` and expects an error containing the cause and "way forward".
- F-tests-2, LOW, design. Location: `internal/shuttleengine/codexengine/sandbox_test.go`. The empty-output failure branch of `runSandboxProbe` (a missing binary or a start failure) and the timeout have no row. Evidence: `TestRunSandboxProbe` rows all use a fake that prints the cause on a non-zero exit. Fix: add a row with a nonexistent `bin` that expects an error containing `sandbox:` and no trailing output. Leave the timeout untested unless it becomes injectable.

**Tool calls**
1. `grep` for `sandbox_mode`, `neither` and `none of` in the codexengine tests. The diff does not show whether tests assert the old `LoadConfig` error text or the invalid-value row.
