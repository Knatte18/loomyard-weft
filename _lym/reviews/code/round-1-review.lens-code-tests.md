```yaml
lens: code-tests
outcome: reported
```

**Lens: code-tests. Per behaviour, the test that pins it:**

1. **Plan prompt fills `subagent_model` from `cfg.SubagentModel`:** pinned by `TestPlanTable` (`plan_test.go:139`), which expects `model: "haiku"` in the lead prompt. `planTableConfig` supplies `haiku`, so a broken fill fails the row.
2. **Discussion-Write chair prompt fills `subagent_model`:** pinned by `TestDiscussionTable` (`discussiontable_test.go:188`), which adds `model: "haiku"` to `chairWants`. The config helper sets `haiku`.
3. **Darn prompt fills `subagent_model`:** pinned by `TestDarnSpec` (`spec_test.go:87`), which expects `model: "haiku"` in the prompt.
4. **`DarnTable` refuses an empty `SubagentModel`:** pinned by the `empty SubagentModel` row in `TestDarnSpec_Refusals` (`spec_test.go:199`).
5. **`LoadConfig` refuses an empty `subagent_model`, with the `--set` way forward:** pinned by the `empty subagent_model` row in `TestLoadConfig_Refuses` (`config_test.go:551`). It asserts both message fragments.
6. **Template default is `sonnet`:** pinned by `templateConfig()` (`config_test.go:102`), which carries `SubagentModel: "sonnet"` for the template-vs-struct comparison. This relies on the existing test that compares the loaded template with `templateConfig()`; I did not open it.
7. **Key classified in `ConfigSpec` (`subagentModelKey: none`):** pinned by the existing `TestModules_EveryTemplateKeyClassified` (per PATTERN-config-layers), which fails if a template key is unclassified.
8. **`wire()` passes `loomCfg.SubagentModel` to `wireDarn`, then `DarnInputs.SubagentModel`:** partly pinned. The test at `internal/loomcli/wiring_test.go:1064` calls `c.env.DarnTable` after a real `wire()`. If the pass-through were dropped, the table would be built with an empty `SubagentModel` and refused (`spec.go:77`), so the test would fail. Nothing in `loomcli` asserts the configured value reaches the prompt, so a wrong non-empty value would not fail. This is acceptable economy: the end-to-end value flow is covered at the engine level by items 1–3.
9. **Discussion-Write stencil guidance (optional, no fork, batch in one message):** static prose, exempt.
10. **Test economy:** the new `model: "haiku"` assertions pin a marker fill from config, not static prose, so they are not a "new assertion on a stencil's static prose" finding. No new test duplicates existing coverage; every change is a row or an assertion in an existing test.

**Findings:** none.

**Tool calls:**
1. `grep` over `internal/loomcli/*_test.go` for darn wiring and `SubagentModel`: the manifest had no loomcli test coverage of the `wireDarn` pass-through.
2. `grep -n "DarnTable("` over `internal/loomcli/*_test.go`: to see whether a wiring test builds the wired table, which settles item 8.
