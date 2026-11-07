```yaml
lens: code-tests
outcome: reported
```

Lens code-tests: for each behaviour the diff adds or changes, the test row that pins it, or none.

**Pinned**
- Fold in order, object merge per key, list join without repeats, later scalar wins:
  - `TestRenderClaudeSettings/NoFileTakesTheEntriesFoldedInOrder` (`internal/heddleengine/settings_test.go:71`).
- File keys that heddle never set stay, and a heddle scalar overwrites the file's:
  - `TestRenderClaudeSettings/FileKeysStayAndHeddleScalarsWin` (`settings_test.go:76`).
  - This row also pins the nested merge, with `pr` kept beside `commit`.
- Removal of a key, a list item and a whole emptied container that the last render set and no entry sets now:
  - `TestRenderClaudeSettings/WhatNoEntrySetsAnyMoreIsTakenOut` (`settings_test.go:81`).
  - This row exercises the record write and the record read.
  - It keeps a Claude-written `allow` list.
- No entry left:
  - `NoEntryLeftRemovesTheFileTheTakingOutEmpties` pins the file removal.
  - `NoEntryLeftKeepsTheKeysHeddleNeverSet` pins the kept key.
- A file holding no JSON object stays untouched, with a warning and `Unwritten`:
  - `TestRenderClaudeSettings/FileHoldingNoObjectIsLeftAlone`.
- The file missing from the exclude list is not written:
  - `TestRenderClaudeSettings/OffExcludedFileIsNotWritten`.
- A missing `.claude` directory is created, and the staged file is named by target name:
  - These ride on the no-file rows and on the updated `LeftoverTempFileIsOverwritten` (`render_test.go:512`).
- `Show` output and the shipped content (attribution, plugins with the scribe marketplace, the full deny list):
  - `TestShippedClaudeSettings`.
  - `ShippedEntriesArePinned` (`store_test.go:173`) pins the shipped slugs and targets.
- Refusal of a non-object text, a list text, and an object target beside a text target on `Set`:
  - Rows at `store_test.go:151-153`.
  - `SettingsOverrideIsNoObject` (`override_test.go:28`) covers `SetOverride`.
- The seam writes the settings file on `Made`, and the clone and worktree-add path renders it at the anchor:
  - `render_integration_test.go:54`, where `Written` includes `ClaudeSettingsFile`.
  - `render_test.go:243` and `render_test.go:313`.
  - `seam_integration_test.go:80` (exclude list only).
- The template list holds the new path, and `SeedWarpExclude` seeds it:
  - `TestConfigTemplate`.
  - `warpexclude_integration_test.go`.
- `AppendExcludePaths` adds a missing path, is idempotent, keeps comments and other keys, and rejects invalid YAML:
  - `TestAppendExcludePaths` (`warpexclude_test.go:21`).

**Not pinned (findings)**

1. **LOW, design** — `settings.go:174` `dropStale` scalar rule. A scalar that heddle set last render and that was changed since (the current value differs from the previous one) is meant to stay. No row covers it. A row with `file` holding `attribution.commit: "mine"` and `earlier` setting `""` would pin it.
2. **LOW, design** — `settings.go:168`, `dropStale` list rule. A Claude-added item in a list that heddle also set, such as `deny: ["Mine"]` next to a stale heddle item, is not covered.
   - Row 3 has a Claude-written `allow` list but no item added to the same list that heddle owns.
   - The focus names exactly this case, "including after a Claude-written permission".
3. **LOW, design** — `settings.go:743-747` and `settings.go:265`, the no-entry paths:
   - **No entry and no record:** it touches nothing, and no row asserts it.
   - **Removing the record on no entry:** no row asserts that the `.rendered.json` record is deleted. A regression that leaves it behind would make a later render wrongly remove keys.
4. **LOW, design** — `settings.go:192`, `settings.go:204`. These paths have no row:
   - A blank settings file.
   - A record that is malformed or holds no object, which `readRecord` turns into nil. It means a corrupt record silently disables stale removal.
5. **LOW, design** — `settings.go:215`, `objectRefusal`. The branches for `.heddle` missing from `Excluded` and for a blocked staging directory have no settings row. Only the `target.File` branch is covered.
6. **LOW, design** — `settings.go:86`, `foldObjects`. An entry whose text holds no JSON object is left out with a warning, for a hand-edited store. No row covers it.
7. **LOW, design** — `render.go:354`, `Show`. No row for a JSON-object target with no entries, which should return "".
8. **MEDIUM, design** — `internal/configreg/configreg.go:156` and `heddleengine.SettingsExcludeReinterpretation`. No test shows that a hub whose `fabric.yaml` sets its own `warp_exclude` gains the path through the config reconcile. The existing `hubreconcile` tests (`ensure_integration_test.go:233`, `ensure_integration_test.go:1094`, `reinterpret_integration_test.go`) only iterate over reinterpretations generically.
   - The rewrite itself is unit-tested.
   - The registry wiring, a reinterpretation ID of the right kind, and the apply-once marker are not pinned for this entry.
   - The decision record asks for this path explicitly.
9. **LOW, design** — `warpexclude.go:87-99`, `AppendExcludePaths`. Not covered: a non-mapping document, a non-scalar `warp_exclude` value, or several paths in one call. Only the single-path call is tested. The rewrite is internal-facing, so this is minor.

Tool calls:
1. Ran `grep` over the configreg and hubreconcile tests for `Reinterpretation`, plus the heddleengine tests for `claude-settings` and `ReadLayers`. My context lacked the existing reconcile coverage and the other heddleengine tests that touch the new target.
