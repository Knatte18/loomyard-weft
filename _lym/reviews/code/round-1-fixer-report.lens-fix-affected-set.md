Lens fix-affected-set: reran the F1 set searches (renderers of the python check, and tests that reach a real engine `Prepare`), and checked F2–F4 and F5–F7 by reading.

**LOW — webstercli smoke tests**
- Changed text: `internal/webstercli/smoke_test.go:43-44` (`realForkSettingsPath`).
- Summary: this helper calls a real `claudeengine` `Prepare`, and its package's test binary has no `claudeengine.LymBinary` stand-in.
- Evidence: the `TestMain` stand-ins were added only in `claudeengine`, `codexengine`, `shuttleengines`, `orchcli` and `shuttleparity`.
- Without a stand-in, `selfexec.LymBinary` returns `ErrTestBinary`, so `Prepare` now returns the "resolve the lym binary" error.
- Impact: the tests that use the helper are gated at tier 3 (`tiergate.Require(t, 3)`), so tier 0 and 1 stay green.
- The same gap applies to the other tier-3 smoke tests that build a real engine: the `smoke_*` files in `seatengine`, `burlerengine`, `shuttlecli`, `orchcli`, `creelcli`, `treadleengine` and `loomcli`. The ones I read use `NewFromConfig` with `ConfigDirs`.
- Fix: set `claudeengine.LymBinary` in the `TestMain` of each package whose tier-3 tests reach `Prepare`, or hand `Prepare` a stand-in through the helper.

No other findings:
- **F1 renderers:** `pythonAllowedCheck` exists only in `claudeengine/settings.go` and `codexengine/guards.go`, and both take the lym path. The remaining bare `lym shuttle` commands are the batching and edit hooks, which the review excluded.
- **F1 stand-ins:** `LymBinary` is set in `claudeengine`, `codexengine`, `shuttleengines`, `orchcli` and `shuttleparity`.
- **F1 `Prepare` callers:** the other `.Prepare(` callers in tests are fakes (`fakeEngine`, `shuttlefake`).
- **F1 live-check callers:** `renderShellGuard` and `guardCommand` are called only by the engines' own tests and manifest code, which pass the path.
- **F2, F3, F4:** each has its test row, and the doc and summary mentions are updated.
- **F5:** `docs/overview.md:717` is updated, and the one other `hook-python-allowed` mention at line 683 needs no change.
- **F6, F7:** done.

Further calls:
1. Grepped `hook-python-allowed` and the live-check names across the repo, to rerun F1's renderer set.
2. Grepped for test packages that build real engines, and for the `LymBinary =` stand-ins, to rerun F1's test-package set.
3. Grepped `.Prepare(` in tests outside the engine packages, to separate fakes from real engines.
4. Read the tier gates and constructors in the webstercli, seatengine and burlerengine smoke tests, to see whether they reach a real `Prepare`.
