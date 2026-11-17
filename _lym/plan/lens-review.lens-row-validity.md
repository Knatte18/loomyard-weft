```yaml
lens: row-validity
outcome: reported
```

Task: row-validity lens review of the plan in `_lym/plan` (cards 1–9 and the overview).

**Findings**

1. **MEDIUM, card 8 (`TestReport`, `TestManifestCmd`): the fixture catalog can't be injected.**
   - `Report(env, name, engines, cfg)` builds its seats through `Seats(env)`, and `Seats` reads the package-level `seatTables` (card 6).
   - No `SeatEnv` field or other seam lets a test put "a catalog of one fixture table" in its place, and the card lists none. As written, `TestReport` runs the real builders, which need seeded stencils, worktree config and the language skills.
   - `TestManifestCmd` has the same gap: `manifestEnv` returns only `(SeatEnv, EngineSet)`, so the catalog stays the real one.
   - `TestManifestCmd` lives in `internal/shuttlecli` and can't use `manifestEngine`, which is a `_test` type of `shuttleparity`. The existing `shuttlefake.Engine.Prepare` (`internal/testkit/shuttlefake/shuttlefake.go:191`) has no hook for writing a manifest. The card lists no fake engine for shuttlecli that writes one.
   - Fix: add a catalog seam, such as a `SeatEnv` field naming the tables, and list a manifest-writing fake engine that `shuttlecli` can reach.

2. **LOW, card 4 (`TestPrepare_WritesTheManifest`, `ErrHookUnrun` rows): one row repeats an existing test, and the other belongs in that test.**
   - The row "an omitted hook with another reason adds a warning" repeats `TestPrepare_UserSetup`'s `OmittedHookAndWarningReachTheWarnings` (`internal/shuttleengine/codexengine/usersetup_test.go:63`, which asserts the warnings at :116).
   - The row "an `ErrHookUnrun` reason adds no warning" is a warning behaviour of `resolveUserSetup` (`usersetup.go:38-40`), not a manifest behaviour. It belongs as a row of `TestPrepare_UserSetup`, which card 4 doesn't list under `**Edit:**`.
   - Minor: the engine has no "warn log". The tests read a `warnings` buffer (`prepare_test.go:263`).

3. **LOW, card 3 (`TestPrepare_WritesTheManifest`): helpers are unlisted, and the plugin's scope is unstated.**
   - The rows seed a global CLAUDE.md, an installed plugin and a user skill into `claudeDir`. Helpers for this already exist: `writePluginInstalls`, `writeUserSetupFile` and `writeSkillFile` (`internal/shuttleengine/claudeengine/usersetup_test.go:36,54,66`). Card 3 lists none of them under `**Uses:**`.
   - The row doesn't say how the session scope turns the plugin on. The `plugin-context` assertion depends on that: `pluginSessionStartContexts` lists only the plugins the scope enables (`pluginhook.go:126`).

4. **LOW, cards 2 and 3 (plugin-context delivery under the no-runner path): an implementer could easily mark the item `omitted`.**
   - With a nil `RunHook`, `pluginSessionStartContexts` reports every hook as `OmittedHook{Reason: "not run"}` (`pluginhook.go:151`).
   - Card 3 builds Claude's `plugin-context` items from that path and calls them `native`, but no row asserts that the Claude record's `Omitted()` verdict stays empty. If an implementer turns those omitted hooks into `omitted` items, every Claude run record's verdict fails and no planned row catches it.
   - Fix: add an assertion to card 3's autonomous row that `Omitted()` returns no plugin-context item.

**Tool calls**
1. `internal/shuttleengine` `fakeEngine.PrepareHook`, `findRunByStrand` and `TestSweepOrphans`: to check card 2's rows. The context held only their glyphs.
2. `codexengine` `fakeUserSetupSource`, the warning seam and `resolveUserSetup`'s omitted-hook warning: to check card 4's `ErrHookUnrun` rows.
3. `OmittedHook`, `HookRunner`, `pluginSessionStartContexts`' omitted paths and `engineWithAnchor`: to check card 3's plugin and fixture rows.
4. claudeengine test helpers and the Prepare test list: to check card 3's fixture needs against existing helpers.
5. seatengine's `seatfake_test.go`, `HoldDeferral` uses, and shuttlecli's `TestOpenCmd_ListModels` and `TestCommand_PositionalArgValidation`: to check the receivers of cards 6 and 8.
6. `shuttlefake.Engine` and `newCodexLaunchFixture`: to check card 8's fake engine and card 5's fixture.
