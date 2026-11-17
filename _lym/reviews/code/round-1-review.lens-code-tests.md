```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, name the test that would fail if it broke.

**Per behaviour, the test that pins it:**

1. **`branch_prefix` is classified `CategoryMachine`, so fabric's `MachineKeys` is `[branch_prefix]`.** Pinned by `internal/configreg/configreg_test.go:196-198`, the new `wantMachineKeys` row for fabric in `TestModules_EveryTemplateKeyClassified`. Reverting the category to cross-pair fails it.
2. **`fabricengine.LoadConfig` stacks the machine override over the hub file and keeps the hub file's `pathspec`.** Pinned by the new subtest in `TestLoadConfig` (`internal/fabricengine/config_test.go:116-137`). Without a machine key, `configengine.LayerFile` (`layers.go:146`) stacks no machine file, the prefix reads `hub/`, and the assertion fails. The redundancy report lists `TestLoadConfig` as kept, and no other test asserts this value.
3. **The local layer still refuses `branch_prefix`.** Pinned indirectly by the same configreg test: the `wantLocalKeys` check (`configreg_test.go:173-175`) fails if `branch_prefix` were ever classified `CategoryNone`. Any non-None category is local-denied by `LocalDenied` (`layers.go`), and the configengine local-refusal tests enforce that for every non-None key. No fabric-specific row writes `branch_prefix` to a local file. That is acceptable, because the generic rule plus the classification pin covers it.
4. **`lym config fabric --layer machine` is no longer refused.** `selectedLayer` (`internal/configcli/hubwide.go:117-120`) decides this purely from `MachineKeys`, which row 1 pins for fabric. The machine-layer write path for a hub-wide module is pinned by the gate rows in `TestDispatchLayers` (`configcli_test.go:1573-1696`). There is no fabric-specific row, and none is needed.
5. **The refusal of `--layer machine` on a hub-wide module with no machine key still holds.** Pinned by the repointed `slack` row (`configcli_test.go:1687-1692`). Slack's only key is `CategoryNone` (`internal/slackengine/config.go:41-43`), so `MachineKeys` is empty and the `hubwide.go:119-120` branch fires with `"lym config slack"`. The row still asserts that no file is written.
6. **A fabric machine file holding `pathspec` or `warp_exclude` is refused.** This runs through the generic `ValidateMachine` path, pinned by the gate row "machine layer refuses a key the layer closes naming it and leaves no file" (`configcli_test.go:1619`). The behaviour is driven by category, not by module, so no fabric row is needed.
7. **The doc and comment changes** (`doc.go` files, `layers.go` comment, `template.yaml` comment, `docs/overview.md`, the `pattern/` file) are static prose and exempt.
8. **The widened `TestLoadConfig` doc comment.** The claim "a local pathspec or warp exclude list is refused" describes the existing loop (`config_test.go:139-165`), which predates this diff. The only new claim, "a machine override's branch_prefix wins", matches the new subtest. No test-economy issue: the new subtest is unique, and neither `TestDispatchLayers` nor `TestModules_EveryTemplateKeyClassified` asserts the loaded prefix.

**Findings:** none. Every behaviour the diff adds or changes has a test that fails if it breaks.

Tool calls:
- Read `internal/configreg/configreg_test.go:150-215` and grepped tests for local `branch_prefix` rows. My context lacked the `wantLocalKeys` and `LocalKeys` assertions that pin the local refusal of `branch_prefix`.
