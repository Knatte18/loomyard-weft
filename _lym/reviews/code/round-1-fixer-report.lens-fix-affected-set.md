```yaml
lens: fix-affected-set
outcome: reported
```

Task: check the fix against F2–F5, the findings whose premise is a set of sites or tests, and re-run each finding's own search.

No findings.

Every set the fix had to cover is covered:

- **F2 (`commandDeps` Title sites):** All three `CommandDeps{` literals set `Title` or never reach a `{title}` placeholder.
  - Production: `commandDeps` at `internal/battencli/wire.go:830` sets `Title`.
  - `internal/testkit/envkit/envkit.go:565` sets `Title`.
  - The `battencli` tests at `continue_pass_test.go:111` and `reopen_window_test.go:62` leave `Title` nil, but their commands are `open-window {slug}`, which has no `{title}`. `Title` is called only when the command holds `{title}`, per `shedadapters/command.go:120`.
  - The new `TestCommandDeps_TitleOpensTheSlugWithTheTasksHandle` covers the wiring.
- **F3 (kept-strand sites):** Both strand-keeping sites are documented. The status strand's is in `loomcli/doc.go:62` and the runs strand's is in `orchcli/cli.go:102-104`. No other strand's title uses `TitleWithHandle`.
- **F4 (status spec joins):** `ensureStatusStrand` at `sharedbootstrap.go:301` is the only production caller of `statusStrandSpec`. `statusStrandAddSpec` is still called directly only by `smoke_drivingsurface_test.go` and `bootstrap_test.go`. Those are test-only fixtures and are not the wiring the finding named.
- **F5 (runs-title stubs):** `runsWindowTitle` is stubbed in one place, `start_test.go:168`, now with `harnessRunsWindowTitle`. The only other reader of the old `"beam: prime"` stub is `wantDisplay`, which now uses the constant. The remaining `"beam: prime"` hits are independent tests of `runsWindowTitleOf` and the reedengine window-order tests, not the stub. `runstitle_integration_test.go` calls the real func.

Tool calls made:
1. `grep` for `runsWindowTitle`, `harnessRunsWindowTitle` and `"beam: prime"` across `internal`. This reruns F5's search, and the context had no cross-package view of stub sites.
2. `grep` for `statusStrandAddSpec(`, `statusWindowTitle(`, `statusStrandSpec(`, `CommandDeps{`, `{title}` and `.Title()`. This reruns the F2, F3 and F4 searches, for the same reason.
3. `sed` of `envkit.go:560-580`, `continue_pass_test.go:105-120` and `reopen_window_test.go:58-72`. This checked whether the other `CommandDeps` literals leave `Title` nil in a way that matters, and the context did not show them.
