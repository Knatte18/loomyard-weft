```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what the diff's changes force outside the lines it touches.

**Set per change**
- Trust sentence, flag dir and flag-name fragment moved into `shuttleengine/creelhooks.go`: the full set is covered. The only production users are `claudeengine` (`settings.go:124`, `settings.go:465`, `claudeengine.go:272`) and `codexengine`. The tests that used `claudeengine.CreelFlagDir` were updated (`hubgeom_test`, `claudeengine/command_test`, `claudeengine/settings_test`). The claudeengine docs were reworded. No other caller turned up in the grep.
- `IdleWaker` capability and `wakeSession`: covered. `hookwait.go` is the only consumer. `shuttleengine/doc.go` and the hook-wait `Long` help were updated. `shuttleengines` registers `codex` as a renderer, and `TestToolBoundaryRenderer` was extended.
- `codexengine` marked `CreelHooks`: `deliver.go:242` and `open.go` read the mark generically, so the daemon needs no change. The `codexengine` package doc, the `Prepare` and `OpenLaunch` comments, `creelengine/doc.go`, `README.md` and `docs/overview.md` were all updated. `shuttlecli/open.go` and `open_test.go` still say opened Codex sessions have no creel hooks, which stays correct.
- Prompt-file text now carries the trust sentence: the `prepare_test` rows were updated. `open_test.go` is unaffected because `OpenLaunch` writes no prompt file.
- `hookSettings` signature gained a `creel` argument: every caller was updated (`Prepare` and the `prepare_test` and `command_test` call sites).
- `PATTERN.md` and `pattern/`: the diff touches neither, and no declared PATTERN change is listed in my context. `PATTERN-shuttle-provider-seam` is respected, since `creelcli` reaches Codex only through the `IdleWaker` interface. `PATTERN-child-processes` is met, since `WakeIdle` logs the spawn and waits with `CombinedOutput`.
- Commit messages: no `Pattern-Departure:` trailers are present, and none are needed.

**Findings**

1. **LOW, consistency**, `internal/creelcli/hookwait.go:18`: the `hookWaitVerb` comment still reads "which the Claude engine's waiter hook spells". The Codex engine now spells the verb too, as `creelWaitCommand` in `codexengine/command.go`. The diff reworded the neighbouring `hookWaitCmd` and `hookWait` comments but missed this one.
2. **LOW, consistency**, `internal/creelcli/hookread.go:19`: the `hookReadVerb` comment says "which the Claude engine's hooks spell". Codex's `creelMailCommand` now spells it as well (`command.go`). This is the same stale-wording gap as finding 1.

**Tool calls (3 of 6)**
1. grep for Codex wording near creel, hook and mail in the creel docs, `shuttlecli`, `contracts`, `docs` and `pattern`. My context did not cover those spots outside the diff.
2. grep for Claude-only wording near the generalized waiter and for `ToolBoundaryContextRenderer` users. This was needed to find stale comments and consumers.
3. `git diff --stat` on `PATTERN.md` and `pattern/`, the Claude trust-sentence tests, and a listing of the `codexengine` directory. This confirmed no PATTERN change and that no `codexengine` file was missed.
