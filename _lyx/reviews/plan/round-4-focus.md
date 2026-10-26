---
round: 4
focus:
  - "Card order around the removal of `lyx orch start`: does any card that removes or renames a CLI verb leave a shipped file or tier0 test that names that verb (`internal/shortcutcli/shortcuts.yaml`, `internal/shortcutcli` `TestMount_File`, `cmd/lyx` `TestShortcuts_FileAndRootListing`) failing before the card that rewrites it?"
  - "Test tiering in the pawl engine card: does every test file the plan adds carry exactly one `tierN` tag, with only support files untagged, as `PATTERN-test-speed` and `cmd/lyx/tiertags_test.go` require?"
  - "Shared fixtures that enumerate every seam section: for each card adding a seam to `shedrecipe.Env` or `landingshed.Deps`, are `internal/testkit/envkit` `FullEnv` and `nilLegal`, both `intentionallyNil` tables and `TestLandingDeps_EveryFieldPopulated` either targets of that card or untouched by it?"
---

Each entry names a place where a card's own gate can fail on a file another card owns.
