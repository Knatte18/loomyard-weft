```yaml
lens: row-validity
outcome: reported
```

Lens row-validity: whether each planned row reaches its branch, would fail if the code were wrong, and is needed.

**MEDIUM — Card 2, D6 seed criterion: the cited load row does not reach the local layer.**
- Card 2 cites `internal/loomengine#TestLoadConfig_Loads`' row "empty fan keys never read a malformed lenses.yaml" as the step where a seeded pair loads `code_reviewer_fan: ""` as `lenses.FanList{""}`.
- That row puts the value through `values` into the seeded base file (`internal/loomengine/config_test.go:168-176`), not the pair's local file.
- Seed-Child writes the profile block into the local layer, and that layer is the path card 1 opens. No planned row loads a local `code_reviewer_fan: ""` through `LoadConfig`.
- Card 1 only removes `code_reviewer_fan` from `TestLoadConfig_Refuses`' local-refusal loop (`internal/loomengine/config_test.go:685-700`). It adds no admitted `LoadConfig` row.
- `TestModules_LocalLayerRefusals`' new admitted row stops at `configengine.Layers` and never decodes the `FanList`.
- Fix: add a row to the layered subtest of `internal/loomengine#TestLoadConfig_Loads` (`internal/loomengine/config_test.go:439-470`), beside "a local driver_kind go is open to the local layer".
  - `local: "code_reviewer_fan: \"\"\n"`
  - `mutate: func(c *Config) { c.CodeReviewerFan = lenses.FanList{""} }`
- Put that row on card 1 (`Edit`) and cite it from card 2 in place of the base-file row.

**LOW — Card 1, `TestModules_LocalLayerRefusals`: the new refused row repeats an existing test.**
- The new row `{"loom", "code_fixer_fan: \"\"\n", "code_fixer_fan", false}` repeats a behaviour already reached.
- `TestLoadConfig_Refuses`' subtest "a local fan key is refused naming the local file and the key" keeps `code_fixer_fan` in its loop (`internal/loomengine/config_test.go:687`). It asserts the same `ErrInvalid` naming the local file and the key.
- Fix: drop the refused row and keep only the admitted `code_reviewer_fan` row.

**LOW — Card 4, `TestTemplateChecks_FlagBrokenTemplates`: one `writerFanGaps` row repeats the template call.**
- The row "the template's pairs with `repo-invariants` reviewer-only give none" repeats what `TestConfigTemplate_DecodesAndIsSelfConsistent` does once card 4 replaces its inline check.
- That test calls `writerFanGaps` over the template's own orders with `reviewerOnly` holding `repo-invariants` and fails on any message.
- Fix: drop that row and keep the three rows that must fail or pass on hand-built input.

**LOW — Card 5, csharp `review-code` exact-order assertion: the existing layer checks already catch it.**
- The planned assertion that `ResolveFan(review-code)` under `lens_set: csharp` is exactly `code-correctness`, `code-affected-set`, `code-tests`, `numerics` repeats checks the layer subtest already makes.
- A layer fan must start with the base fan, and an extra must not repeat a base entry (`internal/lenses/config_test.go:412-426`).
- The fan must end with `numerics` (`internal/lenses/config_test.go:431-439`).
- `CheckLoad(lens_set: csharp)` resolves every fan, so a leftover `code-pattern-fit` fails as an undefined lens (`internal/lenses/config_test.go:378-380`). Card 3's template no longer defines it.
- Fix: drop the extra assertion.

Further tool calls:
1. Read `internal/shedrecipe/entries_burlerhalves_test.go` for `TestBurlerReviewEntry_ClusterFan` and its "EmptyStringSettingRecordsASoloRound" row, and the head of `TestLoadConfig_Loads` with its local-file helper. My context did not hold the row card 2 cites or whether loom's load tests drive a local file.
2. Read `internal/loomengine/config_test.go:438-500`, the layered subtest. I needed to confirm it takes a `local` field and compares the whole `Config`, so a `FanList{""}` row fits there.
