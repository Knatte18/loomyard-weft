```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, the test that pins it, or none.

**Behaviour → pinning test**

- Prime green is `#1b5e20` and is `palette[0]`: `TestPaletteStartsWithThePrimeGreen` (`color_test.go`). Pinned.
- `PickBeamColor` (skips green and held colours, case-insensitive, all-held fallback): `TestPickBeamColor`. Pinned.
- `PickWorktreeColor` (project exclusion, siblings, keeps current when allowed and free, drops it otherwise, palette exhausted): `TestPickWorktreeColor`. Pinned.
- `SiblingWorktreeColors` (same project colour only, skips self, old settings with no activity colour): `TestSiblingWorktreeColorsTakesSameProjectActivityColors`. Pinned.
- `ActivityBarColor`: `TestActivityBarColor`. Pinned.
- `Foreground`, including the 0.179 threshold (`#757575` white, `#808080` black) and a non-colour value: `TestForeground`. Pinned.
- The inactive-text `aa` alpha, the nine written keys, and no status-bar, border or tab keys: `assertLook` in `config_test.go`. This is inferred from its comment ("exactly lym's … keys"), not read in full.
- The settings writer keeps other keys, rewrites its own, and leaves undecodable files byte-identical (`{not json`, `[1, 2]`, `null`): `TestWriteConfigRewritesOwnSettingsKeysAndOverwritesTasks` and `TestWriteConfigLeavesUndecodableSettingsAlone`. Pinned.
- `lookFor`, covering prime, beam, an unrecorded beam, a task under a beam, siblings avoided, and the done beam ignored: `TestLookFor` in `look_test.go`. Pinned.
- A hub with no shortname falls back to the slug: `TestLookForWithoutShortnameTitlesTheSlugAlone`. Pinned.
- Failure paths of `lookFor`, an unreadable board and an unresolvable parent, each falling back to a task under the prime: none.
- `SetOwners` records `BeamColor` from the held colours and keeps a colour once recorded: `TestSetOwners` in `ownerset_integration_test.go`, the `held` and `repick` cases. Pinned.
- `beam_color` dropped from a non-beam entry in `migrateEntries`: the `only a beam project keeps its owner and colour` row in `migrate_test.go`. Pinned.
- `normalizeTask` dropping `beam_color`: none. `legacy_test.go:71` calls `normalizeTask` but has no `BeamColor` assertion, and `migrateEntries` is a separate site.
- `OwnerEntry.BeamColor` through `ownersOf`: `owners_test.go:110-117`. Pinned.
- The `replayFields` `beam_color` entry: none. No `BeamColor` reference in any replay test.
- `beam_color` surviving load, save and replay as an acceptance criterion: only the migrate and owner-set rows exist, and no replay or round-trip row.
- `lym beam create` re-runs the owner step for a beam with no colour: the `BeamRecordedHereWithNoColourRunsTheOwnerStepForOne` row in `TestCreate`. Pinned.
- `lym beam create` passes `BeamColor: vscode.PickBeamColor`: no test asserts the picker is passed. `createDeps` fakes `SetOwners`, so dropping the field would break nothing.
- `lym ide seed` with no slug seeds the prime (`cli_test.go:108`) and a slug still works (`cli_test.go:90`). Pinned.
- `lym ide open` without a slug still gives a usage error: the `open without a slug` row. Pinned.
- The prime's name accepted as a slug: none specific. `seed loomyard` is not tested, though `cli_test.go:108` covers the same path through `WorktreeName`.
- A beam's workspace file splices the beam's own settings, else the prime's: `SplicesTheBeamsOwnSettingsIntoItsFileAndThePrimesWhenItHasNone` in `hubworkspace_integration_test.go`. Pinned.
- Failure path of `beamWorkspaceSettings`, an unreadable settings file or anchor: none.
- `window.title` in the workspace file matching the folder's: only implied by the splice row.
- `Spawn` and `Seed` write the new look into settings: `spawn_clean_integration_test.go:139` and `spawn_hub_integration_test.go`, updated. Pinned.

**Findings**

1. MEDIUM, design. The `beam_color` replay field is untested.
   - Location: `internal/boardengine/replay.go:57`.
   - Summary: the added `replayFields` row has no test. Dropping it would silently lose a colour recorded by a concurrent replayed write.
   - Evidence: grep of `internal/boardengine/*_test.go` shows `BeamColor` only in `migrate_test`, `owners_test` and `ownerset_integration_test`.
2. LOW, design. The `normalizeTask` drop of `beam_color` is untested.
   - Location: `internal/boardengine/task.go:279`.
   - Summary: only `migrateEntries` has a row for dropping the field.
   - Evidence: `legacy_test.go:71` calls `normalizeTask` with no `BeamColor` assertion.
3. LOW, design. `beamWorkspaceSettings` has no failure-path test.
   - Location: `internal/fabricengine/hubworkspace.go:62-77`.
   - Summary: a beam whose anchor or settings file cannot be read makes `RegenerateHubWorkspace` fail. This is new behaviour, and no test exercises it.
   - Evidence: `hubworkspace_integration_test.go` adds only the splice row.
4. LOW, design. The `lookFor` degradation paths are untested.
   - Location: `internal/ideengine/spawn.go` (`lookFor`, the `beamColors` and `parentOf` error branches).
   - Summary: nothing covers a board read error or a `ParentOf` failure. Both should fall back to a task under the prime with a warning.
   - Evidence: `look_test.go` has only `TestLookFor` and the no-shortname test.
5. LOW, design. Nothing asserts that `lym beam create` passes the colour picker.
   - Location: `internal/beamcli/create.go:164`.
   - Summary: the fake `SetOwners` in `createDeps` ignores `BeamColor`, so removing the field would pass every test.
   - Evidence: the new row in `create_test.go` asserts only the call list `{"owner","start"}`.
6. NIT, design. Slight test overlap.
   - Location: `internal/ideengine/look_test.go` (`TestLookFor` rows t3 to t5) and `internal/vscode/color_test.go` (`TestSiblingWorktreeColorsTakesSameProjectActivityColors`, `TestPickWorktreeColor`).
   - Summary: sibling avoidance is exercised at both layers. The `look_test.go` rows also cover the wiring from `lookFor`, so I judge them non-redundant and raise no finding.

No new test duplicates an existing test's coverage. The deleted `TestPrimeResolveLogLevel` went with its removed code.

**Tool calls**
1. `git diff … -- '*_test.go'` filtered to the test names and rows: the manifest held no test files.
2. `grep` for `BeamColor` and `beam_color`, `normalizeTask`, `ParentOf` and the failure cases, and the CLI slug cases in the test files: to confirm which behaviours have a test.
