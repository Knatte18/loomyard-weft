```yaml
lens: row-validity
outcome: reported
```

Lens: row-validity. I checked whether each planned test row can reach its branch, would fail if the code were wrong, and is needed.

**Findings**

1. **MEDIUM — Card 5, `TestNewWithSeams_CodexTakesTheGivenSeams`: the probe row cannot reach the probe on the fixture it names.**
   - The card runs on `newCodexLaunchFixture` "under codex.yaml's `auto` sandbox mode". That fixture writes `sandbox_mode: danger-full-access` (`internal/shuttleengines/codexlaunch_test.go`, inside `newCodexLaunchFixture`).
   - `launchSandboxMode` returns before calling the probe under `danger-full-access` (`internal/shuttleengine/codexengine/sandbox.go`, `Codex.launchSandboxMode`). The "probe called" assertion fails unless the test rewrites the worktree layer.
   - The fixture's config directory also holds no plugin, so the hook runner is never called either.
   - The card lists the fixture under `Uses`. Either list `internal/shuttleengines#newCodexLaunchFixture` under `Edit`, or state that the test writes the `auto` layer and a plugin with a session-start hook (an `installed_plugins.json` and a `hooks/hooks.json`) itself.
   - LOW: the new test reuses the fixture and Prepare path of `TestNew_CodexLaunchCarriesTheClaudeUserSetup` and could be a second case of it.

2. **MEDIUM — Card 8, `TestManifestCmd_RefusesAMissingOrUnknownSeat`: the unknown-seat row cannot get past `PersistentPreRunE` at tier0.**
   - Run through `clihelp.Execute(Command())`, the verb first needs `lymcwd.Resolve`, `hubgeom.ReedGeometry` and `hubgeom.PauseGeometry` to pass (`internal/shuttlecli/cli.go`, the `PersistentPreRunE` in `Command`). That needs a git checkout inside a hub, and a tier0 test may not spawn `hubforge`.
   - The card names no such fixture.
   - Existing verb tests avoid this by building `shuttleCLI` by hand and running the subcommand directly (`internal/shuttlecli#TestOpenCmd_ListModels`). The unknown-seat row should do the same.
   - The no-argument row does work: cobra checks positional arguments before the pre-run. It belongs in the existing `TestCommand_PositionalArgValidation` table (`internal/shuttlecli/cli_test.go:240`), not in a new top-level test.

3. **MEDIUM — Card 8: no row reaches the verb's own non-zero exit naming a missing item, or its pass with contexts listed unrun.**
   - `TestReport` covers `Report`, but no row runs `manifestCmd`'s branch that prints `output.Err` with `<channel> <key>` and exits non-zero (D10's acceptance criterion).
   - A hand-built `shuttleCLI` row, as in finding 2, can reach it if the engine set can be injected into the verb. The card builds the set inside the verb from `NewWithSeams`, so it is not injectable as written.

4. **MEDIUM — Card 6, `TestSeats_CodexManifestHoldsClaudes` / `Seats`: the stand-in inputs leave two builders unable to build.**
   - `PartsPlanTable` reads the parts file from the anchor (`internal/loomengine/partsplan.go:62`, `readPartsFile(anchor)`).
   - `DescribeTable` refuses an empty `DecisionRecordPath`, `TaskBranch`, `ParentBranch` or `MergeBase` (`internal/landingshed/describe.go`, the `required` list in `DescribeTable`).
   - The card says only "stand-in inputs: … a placeholder slug and paths under env.Dir". It does not say the fixture writes a parts file or fills those fields, so the guard test fails on a builder error rather than on parity. That leaves a fact for the implementer to discover while working.

5. **MEDIUM — Card 3, `TestPrepare_WritesTheManifest`: the native user-setup items get no row.**
   - The rows assert guards, skill loads, notices, standing passages and settings.
   - No row asserts the native `instruction-source` items, `skill-listing` or `plugin-context`. Those are the Claude items Codex is compared against.
   - Dropping them from `claudeManifest` would fail no row of this card. Card 6's parity test would also stay green, because `MissingItems(claude, codex)` only finds Claude items Codex lacks, so a lost Claude item reads as parity.
   - `engineWithAnchor` sets no Claude config directory either, so such a row also needs one (`claudeDir` or `CLAUDE_CONFIG_DIR`), which the card does not state.

6. **MEDIUM — Card 4, `TestPrepare_WritesTheManifest`: the "a skill no root holds gives no `skill-load` item" row cannot fail if the code is wrong.**
   - No row asserts that a skill a root does hold gives an item, so a `codexManifest` that never emits `skill-load` passes.
   - The same gap holds for `standing-passage`, `instruction-source` (read directly vs pointer, decided by `claudeFileIsUnread`) and `batching-nudge`: no row asserts them.
   - The plugin rows need the existing fake `internal/shuttleengine/codexengine#fakeUserSetupSource` (`usersetup_test.go:21`), which the card does not list under `Uses`.

7. **LOW — Card 2: the new `fakeEngine.manifest` field repeats an existing seam.**
   - `fakeEngine.Prepare` already calls `PrepareHook(runDir)` (`internal/shuttleengine/fakes_test.go:342`), so a test can write the record through the hook.
   - Using the hook instead keeps the fake unchanged and drops `fakeEngine` from the card's `Edit` list.

8. **LOW — Card 2, `TestSweepOrphans_AgeGuardAndLiveGuid`: this test is not a table.** It is one scenario with a `reflect.DeepEqual` on the removed list (`internal/shuttleengine/rundir_test.go:225`). "Gains a row" here means a seeded `manifests` directory, aged past the guard, plus a survival assertion. The `removed` expectation stays as it is, so the row does fail if the skip is missing.

9. **LOW — Card 6, `TestSpecs_MatchTheStartedSpecs`: compare `HoldDeferral` by nil-ness, never by value.** `fakeShuttle.StartGated` records the started spec by role (`internal/seatengine/seatfake_test.go:200`). The lead's spec carries a non-nil `HoldDeferral` func, so a whole-struct `reflect.DeepEqual` against `Specs` fails unless that field is cleared first. The card says "`HoldDeferral` aside" but not how.

**Tool calls**
- Grepped `fakeEngine`, `newFixture` and the user-setup test names, because the context lacked where the fake and those tests live.
- Read `TestCommand_PositionalArgValidation` and the `seatfake_test.go` index, because the context lacked how verb tests and the seat fake record specs.
- Read `TestOpenCmd_ListModels`, because the context lacked how shuttlecli tests get past `PersistentPreRunE`.
- Read `fakeEngine.Prepare` and the two run-directory tests, because the context lacked their shape for the planned rows.
- Read `DescribeTable`'s required inputs, the `readPartsFile` call and the Codex fake setup source, because the context lacked what the stand-in inputs must supply.
