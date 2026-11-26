```yaml
lens: affected-set
outcome: reported
```

Task: run the affected-set lens over cards 1–14, listing for each change who reads or asserts it and which cards cover them.

**Affected-set list (per change: set found → covering cards)**

- **Card 4: flat loom and tapestry key names to nested paths (templates, decoders, recipes, profiles, fixtures).**
  - Production readers: `loomengine`, `reviewbudget`, `loomcli`, `shedrecipe`, `shedbuild`, `loomrecipe`, `runprofile`, `beamcli#mergeParent`, `hardencli#commitSandboxConfig`, `configreg#CheckSettingKey`/`SettingLookup`, the `contracts/recipes/*` setting refs, and `loomshed/raddlewrite.go`. That last one gets its key from loomcli's deps and only names it in comments. All are covered by card 4, plus card 10 for loomshed's text.
  - Tests that write or read flat keys against a template: configcli `TestConfigOverRealHub` and `TestConfigReconcileInGitRepo`; configreg `TestModules_EveryTemplateKeyClassified`, `TestModules_LocalLayerRefusals` and `TestLookup`; hubreconcile `TestEnsure_CarriesATemplateChangeIntoASeedOnlyConfigFile`; beamcli `TestHubLandDeps_…` and `TestMergeParent`; plus configsync and runsettings tests. All are card 4 members.
  - Tests that use flat names as opaque data and never load them against a template, so they don't break: hubforge `TestNewHub` (`hub_test.go:217`, byte round-trip only), `configengine/changedkeys_test.go:43-46`, `landingshed/finalize_test.go:1006,1051`, `boardengine/store_test.go:205,499`. None is needed.
  - `shuttleengine/requires_test.go:122` and `contracts/stencils/stencils.go:155,161` are covered by card 11 (package glyphs).
- **Card 4: `configengine.KeyAtLine` returns the deepest path.** Its only callers are `loomengine/config.go:880,884` and `loomengine/tapestry.go:216,220`, all covered by card 4.
- **Card 4: `reviewbudget` key builders and `Route` reading leaf paths.**
  - `beamcli/mergeparent.go:83-84` and `loomcli/circling.go:275,278` are covered by card 4.
  - `hardenshed/config.go:192` and `loomcli/hardenwiring.go:343,424` pass hardener's flat keys to `Route`. A leaf-path read still resolves a one-segment path, so nothing breaks there.
- **Card 4: `SettingLookup` and `CheckSettingKey` take a dotted path.** Callers in battencli `wire.go:807,1691`, fabriccli `merge_verbs.go:37` and loomcli pass flat or recipe refs. They are covered by card 4 (recipes), and flat refs stay valid.
- **Card 5: the moved-key refusal turned on.** Every fixture that loads a flat loom key is already moved by card 4. Covered.
- **Card 6: `reconcileHubWide`/`reconcileWorktree` re-signed and `Result.Moved` added.** The callers are in-package `walkHub` and `reconcilePair`, plus configcli `reconcileEnvelope`. Covered by card 6.
- **Card 9: `reconcileWayForward` text.** Asserted by `TestEnsureOrWarn_WarnsWithTheWayForward`, which card 9 covers. Doc text is left to the repo-invariants lens.
- **Card 10: `CheckLocal` re-signed.** Its callers are `loomengine/config.go:860` and `tapestry.go:193`, covered by card 10.
- **Card 10: `ValidateLocal`, `ValidateMachine`, `warnUnknownKeys`, `refuseRetiredKeys`, `refuseHubOnlyKeys` and `CheckSettingKey` texts move to full names.**
  - Test files that match these refusal texts: `configengine/layers_test.go`, `configreg/runsettings_test.go`, `envconfig/config_test.go`, `gateslot/config_test.go`, `shedbuild/build_test.go`, `shuttleengine/codexengine/prepare_test.go`.
  - The configcli, desktophook, fabricengine, hubgeom, ideengine and reedcli hits only matched the phrase "machine override".
  - Card 10 covers configengine through the member `TestValidateLocal` only, so it misses part of this set (finding 1).

**Findings**

1. **MEDIUM**, class design. Location: `_lym/plan/10-loom-key-messages.md`, Edit list.
   - Summary: card 10 rewords `ValidateMachine`'s refusals (`machineRemoval`, the "not in the template" and category refusals) and `warnUnknownKeys`' warning to full names for every module. Yet the only configengine test it lists is the member `configengine_test#TestValidateLocal`. Its rule "the tests inside the packages listed" doesn't reach configengine, which it names only by members.
   - Effect: any `ValidateMachine` or load-path test in `internal/configengine/layers_test.go` that asserts the refusal text is left outside the card's targets.
   - Evidence: the refusal-text grep matched `internal/configengine/layers_test.go`. `ValidateMachine` and its refusals are at `internal/configengine/layers.go:257-300`.
   - Fix: list the configengine tests that assert `ValidateMachine` and `warnUnknownKeys` texts (for example `configengine_test#TestValidateMachine`) as card 10 Edit members, or name `internal/configengine_test#` as a package glyph.

No other gaps: every other change's affected set is covered by its card or by a later card that owns the same text.

Tool calls:
1. Grep of flat loom/tapestry key names outside card 4's package targets: the manifest lacked a repo-wide fixture search.
2. Grep of those names, with lines, in the uncovered files: needed to tell real reads from opaque data.
3. Mapped hit lines to enclosing test functions, plus `KeyAtLine` and `reviewbudget` callers, `raddlewrite.go` and hubforge `hub_test.go`: needed to match hits against card members and find callers the manifest lacked.
4. Grep of tests and docs asserting the configengine and configreg refusal texts: card 10's affected set was not in the manifest.
5. Grep of the refusal-text lines in tests no card lists: needed to tell real assertions from "machine override" phrase hits.
6. Grep of configengine tests for quoted-key refusal assertions: tried to name the exact `layers_test.go` functions; it returned nothing, so finding 1 rests on the file-level match.
