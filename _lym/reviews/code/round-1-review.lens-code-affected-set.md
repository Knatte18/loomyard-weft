```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what the template default change (`model: opus`, `effort: medium`) forces outside the touched lines.

**Per change, the set found**

1. `template.yaml` defaults, `model: opus` and `effort: medium`
   - **Readers of `orch.model` and `orch.effort`:** the only one is `internal/orchcli/start.go:205`, from the `cfg` that `cli.go:199` loads with `orchengine.LoadConfig`. Covered.
   - **Effort vocabulary:** `medium` is in `agentconfig` `effortVocabulary` (`internal/agentconfig/config.go:85`), so `validateEffort` accepts it. Covered.
   - **Template test:** `TestLoadConfig_TemplateResolvesWithNoFile` is the only assertion on the template values, and the diff updates it. Covered.
   - **`smoke_live_test.go:49`:** `smokeOrchConfig` hard-codes `model: ""` and `effort: ""` to run live tests on the provider default. Explicit empty is a valid override, and this is a test fixture, so it is not wrong. Covered.
   - **Docs and stencils:** a search of `.md`, `.go` and `.yaml` outside `docs/research` finds no description of the old empty default for orch. Covered.
   - **`Config` struct comments, `config.go:50-51`:** "empty is the provider default" is still true as the explicit-empty meaning. Covered.
   - **Other templates and golden files:** none reference orch model or effort. Covered.
   - **`MigrateConfig`:** `configreg` references `orchengine.MigrateConfig` but I found no `migrate*.go` under `internal/orchengine`. I did not trace this, and I assume the definition lives elsewhere.

2. `template.go` doc comment: it matches the template. Covered.

3. PATTERN entries and declared changes
   - **`PATTERN-config-layers`:** "a hub base file holds only tuned units" is untouched, since the diff only changes template text.
   - **Reinterpretation rule:** a default change for a key that already exists is arguably not a meaning change, so I raise no finding. The gap below is the reason.
   - **Declared PATTERN changes:** the directive lists none.

4. Commit message: its claim that every start path builds its spec from the loaded orch config checks out. `orchSpec` is the only builder, and it reads `c.cfg.Model` and `c.cfg.Effort`.

**Findings**

- **F-A, LOW, `consistency` or `scope`.** Location: `internal/orchengine/template.yaml:1-2`; evidence: `config.go:93-195` (`LoadOrTemplate` fills template keys only for keys missing from the layers) and `pattern/PATTERN-config-layers.md:109-110`.
  - **Summary:** a hub whose base `orch.yaml` already carries `model: ""` or `effort: ""` from the old template still starts the orch on the provider default, so "every hub and machine start the same way" holds only for hubs without those keys. The record says only that this hub's default layer is already set, so no code change is clearly required. A note or a decision is all this needs.
  - **Caveat:** I did not confirm whether `hubreconcile`'s strip drops template-equal values, or whether old hubs hold these keys at all. I only read the sparse-base rule, so treat this as a possible gap.

Nothing else in the affected set is left wrong.

**Tool calls**
1. A grep for the old-default wording, `effortVocabulary` and orch migrate files. My context lacked the vocabulary definition and whether other files describe the old empty default.
2. A read of `PATTERN-config-layers.md` lines 105-135 and a grep of `orch.yaml` in `configreg`, `hubreconcile` and `configsync`. My context lacked the reinterpretation and sparse-base rules, and I needed to see who writes `orch.yaml`.
