# Card 1 — shuttle.yaml carries the prompt-cache TTL keys

**Create:**
- `internal/shuttleengine#ConfigOpenMaps`

**Edit:**
- `internal/shuttleengine#Config`
- `internal/shuttleengine#LoadConfig`
- `internal/shuttleengine/template.yaml`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/config_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Add the two config keys of D2, carried by `shuttleengine` and never interpreted there (`PATTERN-shuttle-provider-seam`).

- `Config` gains `ClaudePromptCacheTTL string` (`yaml:"claude_prompt_cache_ttl"`) and `ClaudePromptCacheTTLRoles map[string]string` (`yaml:"claude_prompt_cache_ttl_roles"`), placed with the other `claude_*` fields.
  Their field comments say the values are Claude Code's and validated by the engine, not here.
- `template.yaml` gains `claude_prompt_cache_ttl: 5m` and a `claude_prompt_cache_ttl_roles:` map with `driver: 1h` and `webster: 1h`, after `claude_deny_ask_user_question`.
  The trailing comments follow the file's style: the default applies to every role without a map entry; the map is keyed by a strand's role segment (`Spec.Role`), is the operator's own, and replaces the template's map whole when present, so leaving a role out drops it to the default; the values are Claude Code's prompt-cache TTLs, validated when a run starts, and the refusal names the accepted ones.
- `ConfigOpenMaps` returns `[]string{"claude_prompt_cache_ttl_roles"}`, with a doc comment in the shape of `batcher.ConfigOpenMaps`.
- `LoadConfig` passes `ConfigOpenMaps()...` to `configengine.LoadOrTemplate`, so a present map is carried whole and never has the template's entries appended back.
  `LoadConfig` adds no TTL validation: that is `claudeengine`'s, under D4.
- `doc.go`: where it describes the config module, one sentence that `shuttle.yaml` carries the provider's prompt-cache TTL default and per-role map, which `shuttleengine` passes through to the engine uninterpreted.

Tests, in `config_test.go`:
- `TestLoadConfig_TemplateDefaultsResolve` asserts `ClaudePromptCacheTTL == "5m"` and the map equal to `{driver: 1h, webster: 1h}`.
- One new table-driven test, `TestLoadConfig_PromptCacheTTLKeys`, seeds a `shuttle.yaml` per row and asserts both fields exactly:
  a file lacking both keys resolves to the template's default and map;
  a file whose map holds `orch: 1h` and no `driver` resolves to exactly `{orch: 1h}` (an entry the template lacks survives, an entry the template has is not restored);
  a file with `claude_prompt_cache_ttl: 1h` and no map resolves that default with the template's map.
  The seeded files carry the other template keys the way the existing tests seed them, so only the TTL keys vary.
- `test-budget.yaml`: the `internal/shuttleengine` row rises from 140 to 141 for the one new test function, since `cmd/lyx/testbudget_test.go` fails a package over its row (`PATTERN-test-speed`).

**ImpactSummary:** shuttleengine.Config grows two fields and LoadConfig declares one open map; every existing caller compiles unchanged, and nothing reads the new fields until card 3.

**Commit:** 1: shuttle: carry a prompt-cache TTL default and per-role map in shuttle.yaml
