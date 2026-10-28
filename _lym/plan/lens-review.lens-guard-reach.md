```yaml
lens: guard-reach
outcome: reported
```

Task: a guard-reach review of each guard, refusal, branch and test row the plan adds or changes.

**Findings**

1. MEDIUM — Card 1 (D5 bound): a wrong fan name in the local layer never reaches the refusal the bound relies on.
   - D5 says a wrong name "fails the pair's loom config load through `ValidateFanKeys`". The route this card opens is a pair's local `loom.yaml` stacked over the hub base.
   - The only admitted local-layer row is in `internal/configreg#TestModules_LocalLayerRefusals`. It stops at `configengine.Layers`, so it never runs `LoadConfig`, the `OpenMaps` merge or `ValidateFanKeys`.
   - The existing row "unknown code_reviewer_fan" (`internal/loomengine/config_test.go:610`) writes its value into the base document of `TestLoadConfig_Refuses`' table (`config_test.go:585-640`), never into a local file.
   - Card 1 only drops `code_reviewer_fan` from the local-refusal loop (`config_test.go:685-700`). It adds no `LoadConfig` row with a local `code_reviewer_fan`.
   - Two states go unreached through the real loader: a local `code_reviewer_fan: ""` over the base's `["review-code", ""]` loading as `lenses.FanList{""}`, and a local `code_reviewer_fan: nosuchfan` refused naming the key.
   - Proposed fix: in place of the dropped loop entry, add two subtests to `internal/loomengine#TestLoadConfig_Refuses` (or the admitted one to `#TestLoadConfig_Loads`) that use `writeLocalLoomConfig`:
     - `""` loads as `FanList{""}`.
     - `nosuchfan` is refused naming `code_reviewer_fan` and `set the entry to ""`.

2. MEDIUM — Card 2 (D6 seed criterion): the chain to a solo round skips the layered state a seeded pair is actually in.
   - Card 2 argues the chain from three tests: `internal/battencli#TestPrepareRunSettings`, which produces the local document; `TestLoadConfig_Loads`' row "empty fan keys never read a malformed lenses.yaml"; and `internal/shedrecipe#TestBurlerReviewEntry_ClusterFan`'s row "EmptyStringSettingRecordsASoloRound".
   - The `TestLoadConfig_Loads` row puts `""` in the base document (`config_test.go:166-176`). A seeded pair instead holds the scalar `""` in its local layer over a base or template list.
   - No row loads that stacked state, so the criterion's middle step, "loads `code_reviewer_fan` as one empty entry", goes untested for a seeded pair.
   - The row proposed in finding 1 for local `""` also closes this; Card 2 could then name it under `**Uses:**`.

3. LOW — Card 3 (D1 criterion: the exemption still fails any other lens, `leanness` among them): the helper row passes its own `exempt` map, not the one the template check builds.
   - `TestTemplateChecks_FlagBrokenTemplates` drives `plan:internal/lenses#emphasisLineGaps` with a literal `exempt` of `{board-scope}`.
   - The real exempt set, computed in `internal/lenses#TestConfigTemplate_DecodesAndIsSelfConsistent` from the `review-*` and `fix-*` orders plus `board-scope` (`config_test.go:289-303` and `350-358`), is never checked.
   - If that computation came to admit `leanness`, every planned row would still pass.
   - Proposed fix: move the procedure-set computation into a helper that both tests call, or assert in the template test that `procedureLenses` lacks `leanness` and holds `board-scope`.

**Per-guard list**

| Card | Guard or row | Reachable |
|---|---|---|
| 1 | `LocalLayerRefusals` row, `code_reviewer_fan` admitted | yes |
| 1 | `LocalLayerRefusals` row, `code_fixer_fan` refused | yes |
| 1 | `TestPrepareRunSettings` darn `darn_reviewer_fan` and `review_fan` admitted rows | yes |
| 1 | `TestPrepareRunSettings` `webster_fan` and `webster_reviewer_fan` rows | yes |
| 1 | Local-loop refusal for the five remaining fan keys | yes |
| 1 | `ValidateFanKeys` refusal from the local layer | no row reaches it (finding 1) |
| 2 | `TestTemplateLadder` fan presence and absence per block | yes |
| 2 | `TestTemplateDocsProfile` | yes |
| 2 | battencli template-default row | yes |
| 2 | `TestProfilesCheckLoad` template row, which needs card 1 first | yes |
| 2 | Local-layer stacked load of `""` | no (finding 2) |
| 3 | `emphasisLineGaps` rows | yes, over hand-built input only (finding 3) |
| 3 | `TestLoad` tuned `interactions` row | yes |
| 3 | `TestDiscussionTable` `[board-scope]` rows and the dropped-lens skip | yes |
| 4 | `writerFanGaps` rows | yes |
| 4 | `way-forward` emphasis requirement, since `way-forward` now sits in no `review-*` or `fix-*` fan | yes |
| 5 | `code-tests` "Test economy" absence check | yes |
| 5 | csharp `review-code` exact order | yes |
| 5 | `TestCheckLoad` three-lens remove emptying `review-code` | yes |

**Tool calls**

1. Grepped `configengine` for open-map stacking and the setting reader. The context did not show how a local scalar stacks over a base list.
2. Read `internal/loomengine/config_test.go:585-665`. The context did not show which layer `TestLoadConfig_Refuses`' table writes its value into.
