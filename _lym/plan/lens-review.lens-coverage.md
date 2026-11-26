```yaml
lens: coverage
outcome: reported
```

Task: a coverage-lens review of the plan. I listed each decision, acceptance criterion and branch the cards introduce, and checked which card and test row reaches it.

**Findings**

1. **MEDIUM** — D1 acceptance criterion, Card 4. No row checks that the loom and tapestry templates hold no top-level key starting with an area prefix the table moves, and no row checks that the loaded *tapestry* config holds the template defaults at `plan.*`/`parts.*`.
   Evidence: `04-nest-loom-tapestry.md` "Tests" lists only `TestLoadConfig_Loads` (loom defaults) and relies on `TestConfigTemplate_ContainsEveryConfigYAMLTag`. That test pins tags against the template. It cannot fail on a leftover flat key that also has a matching tag. `TestLoadTapestryConfig` gets no defaults row.

2. **MEDIUM** — D4 criterion "Seed-Child's refusal prints one way forward", Card 5. No row runs Seed-Child.
   Evidence: `05-moved-key-tables.md` rows assert only `TestPrepareRunSettings` (no `way forward:`). The wrapper that adds the single way forward is `internal/battenshed/seamchild.go:152-156` (`writeRunSettings`), and no battenshed or battencli test is listed. The criterion names "a task's run_settings at Seed-Child".

3. **MEDIUM** — D4 profile-block site, Card 5. The profile check (`configreg#checkProfiles`, reached by `lym config profiles`) and `lym board upsert` (`boardcli#validateRunSettings`) each get no row.
   Evidence: Card 5 reaches profiles only "through `LayerRunSettings`". The criterion's profile site is the profiles check, and `configreg#TestProfilesCheckLoad` is not on Card 5. D4 says "`lym board upsert` and the profile check refuse the write itself", and no row reaches either.

4. **MEDIUM** — D4 `--set` way forward, Card 2. A `--layer local` or `--layer default` set of an old key would print a way forward without the layer flag, and no card makes the change that Card 2 credits to configcli.
   Evidence: `02-config-moved-keys.md`: "with the layer flag the operator gave kept by the configcli caller's own text". No card lists a configcli production member (Card 5 lists only `internal/configcli#TestDispatchSet`). `configcli#setModule`, `#setLocal` and `#setChecked` stay untouched, and no row sets an old key with `--layer`.

5. **MEDIUM** — D4 matching rule at run_settings, Card 5. `refuseMovedKeys` is new code that must admit `{"loom": {"discussion": {...}}}`, but no row reaches that admitting branch.
   Evidence: Card 5 says the old key `discussion` with an object value "is never refused here, by card 2's matching rule". Card 2's rule lives in `MovedKeyIn` over YAML. `refuseMovedKeys` takes a decoded `map[string]any`, and its only row is `code_max_bounces`.

6. **MEDIUM** — D8 criterion "each perch's budget resolves from `<perch>.max_bounces`, falling back to `review.max_bounces`, for discussion, code, split and plan". The rows sit in `reviewbudget`, which takes values handed in and never decodes a file.
   Evidence: In `internal/reviewbudget#Resolve`, the values come from the caller. A wrong nested tag in `loomengine.Config` or `TapestryConfig`, which `Config.Caps` reads, would pass `TestCheckResolved` and `TestCheckLocal`. The fix is a `loomengine` row that loads a file setting `code: {max_bounces: N}` or only `review.max_bounces`, and asserts `Caps` for each perch, plan included via `WithTapestry`.

7. **MEDIUM** — D8 criterion "…and its Seed-Child applies those budgets" (`lym beam merge-parent`), Card 4. Only the filing is tested.
   Evidence: Card 4's `TestMergeParent` row asserts the filed `run_settings` only. Nothing runs the filed object through `PrepareRunSettings`/Seed-Child into a loaded `Caps(PerchCode)`.

8. **MEDIUM** — D5 "every lym verb … that loads the loom or tapestry config runs `EnsureOrWarn`", Card 9. Three of the four new call sites have no reaching row.
   Evidence: `09-reconcile-before-load.md` adds calls in `beamcli#hubLandDeps`, `webstercli#websterCLI.wireHub` and `shuttleparity#loomConfigOf`. Its only new test drives `loomCLI.arm` (`TestArm_StaleStampReconcilesBeforeANonStartVerbLoads`). The "outside a hub none runs it" branch also has no row.

9. **MEDIUM** — D5 criterion (reconcile a hub base, worktree and local file setting `code_max_bounces: 2` and `discussion: sonnet[medium]`: keep comments, report, second reconcile changes nothing), Card 6. Only partly covered for the hub base.
   Evidence: The `TestReconcileHubWideAt_SeedsFromHubPrimeOrTemplate` row moves only `code_max_bounces`, with no `discussion:` scalar, no comment check and no second pass. Idempotence is asserted only by `TestReconcileAll_Idempotent`, which covers the worktree and local layers, not `ReconcileHubWideAt`.

10. **MEDIUM** — Card 6 branch. `reconcileBaseWorktreeFile`, the standalone full-base path, is told to move old keys, and no row reaches it.
    Evidence: Card 6 Edit lists `configsync#reconcileBaseWorktreeFile`. Every Card 6 row is a hub (sparse) row or a hub-wide row.

11. **MEDIUM** — D7 branch "a walk whose hub-wide write was skipped runs no D7 carry", and the D7 read-failure path, Card 8. Neither has a row.
    Evidence: `08-board-run-settings-carry.md` tests only a failing write through the `beforeRunSettingsCarry` hook. The no-carry-after-skipped-write branch, which `walkHub` gates on `reconcileHubWide`'s skipped result, is unreached.

12. **LOW** — D3 completeness, Card 11. The site list is the exploration's sampled grep, which was explicitly "not exhaustive". Unlike Cards 13 and 14, Card 11 carries no instruction to grep for every module's bare key in Go messages.
    Evidence: The exploration report said "Part 2 is a sampled grep, not a full census". Packages it named with key-bearing messages or tests are absent from Card 11's Edit: `internal/darnengine` (`config_test.go` asserts messages) and `internal/hardenshed`. D3 says "for every module's keys".

13. **LOW** — D4 recipe-build site, Card 5. The row calls `CheckSettingKey` directly rather than building a recipe whose fan or auto-accept reference is a flat name.
    Evidence: `TestSettingLookup` row `CheckSettingKey("loom", "discussion_reviewer_fan")`. The criterion says "at recipe build". `shedrecipe#settingRef`/`fanSetting` reach the lookup at runtime, and only `shedbuild` entry gates call `SettingKnown` at build.

14. **LOW** — D6 lines are mailed with D5's lines, Card 7. No hubreconcile row includes a `profiles.yaml` move line in the mailed notice.
    Evidence: Card 6's `TestEnsure_MovesOldConfigKeysAndMailsTheMoves` seeds only `loom.yaml` files.

15. **LOW** — Branches with no row:
    - `MovedKeyIn`'s parse error marked `ErrInvalid` (Card 2).
    - `moveOldKeys`' `*FileError` and its `dropped … which was null` line at the configsync level (Card 6; only `TestMoveKeys` covers the null drop).
    - `reconcilePair` dropping its lines, and `reconcileHubWide` returning no lines when skipped (Card 6).
    - `KeyAtLine`'s new deepest-key path (Card 4 lists `configengine#KeyAtLine` with no configengine row; a loomengine shape-error row would have to assert `code.reviewer_model`).
    - `CheckSettingKey` descent through a scalar segment (Card 4).

16. **LOW** — Card 4's arras recipe build. "a `loomrecipe` build of the shipped arras recipe passes with `loom.split.reviewer_fan`" names no test function, so it is not a planned row.

Tool calls made: none.
