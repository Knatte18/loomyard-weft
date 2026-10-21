# Card 5 — former-template-values

**Create:**
- `plan:internal/configsync#formerTemplateValues` -> `var formerTemplateValues map[string]map[string]string`
- `plan:internal/configsync#formerValuesDoc` -> `func formerValuesDoc(module string) []byte`
- `plan:internal/configsync#TestFormerTemplateValues_DifferFromTheTemplate` -> `func TestFormerTemplateValues_DifferFromTheTemplate(t *testing.T)`

**Edit:**
- `internal/configsync#stripToSparse`
- `internal/configsync#Result`
- `internal/configsync#TestReconcileAll_MigratesToSparse`

**Intent:**
The former values D1 replaced live in one table, so an old seeded value can be told from a tuned one (D3).
`formerTemplateValues`, declared beside `legacyConfigModules`, maps a module name to its dotted unit to that unit's pre-change template value as YAML text:
- `loom`: `discussion_lenses`, `plan_lenses`, `discussion_reviewer_model`, `plan_reviewer_model`, `webster_fixer_model`, `merge_fixer_model` and every `discussion_`, `plan_`, `webster_` and `merge_` `reviewer_fan` and `fixer_fan` at `""`; `webster_reviewer_model` and `merge_reviewer_model` at `sonnet[high]`; `driver` at `sonnet[medium]`; `discussion_max_bounces` and `webster_max_bounces` at `~`; `webster_auto_accept` at `false`.
- `shed`: `driver_kind` at `llm`.
- `darn`: `darn_reviewer_model` at `sonnet[high]`, `darn_fixer_model`, `darn_reviewer_fan` and `darn_fixer_fan` at `""`.
- `claude`: `prompt_cache_ttl_roles` at the whole map `{driver: 1h, webster: 1h}`.
- `orch`: `soft_threshold_tokens` at `300000`, `threshold_tokens` at `400000`.
- `webster`: `kickstart_pack` at `true`.
- `batcher`: `profiles` at the whole pre-change map, the current one with the `cautious` profile's `budget: 450000`.
An open map is one unit, so `claude.prompt_cache_ttl_roles` and `batcher.profiles` are compared whole.
`formerValuesDoc` renders a module's entries as one YAML document over the module's top-level keys, empty for a module with none, so a strip compares it as a layer below with `yamlengine.Strip` and the module's open maps, the same comparison `stripToSparse` makes with the layers below.
The table holds only the D1 keys' values from immediately before this change; an older template generation's value is kept as tuned.

`stripToSparse`, the strip of an unmarked worktree's full file, also strips each unit equal to the module's former value after the strip against the layers below, and reports those keys in `Stripped` too, whether or not the board's base is migrated.
`ReconcileAll` reaches it only for a worktree without its own `sparse` marker and with every repo default present, under `Migrate` and `Apply` for a write, so it runs wherever `hubreconcile` strips a worktree: the walk, the single-pair call, a pair the migrating walk skipped as mid-merge and a clone's prime; the bound is the worktree's own `sparse` marker, written after the strip, so each worktree is stripped of former values once and a marked worktree's file is never stripped.
A worktree unit equal to a former value is dropped even where the repo default holds another tuned value for it, so the worktree resolves to the default's value, as the decision record's auto-mode assumption records.
`Result.Stripped`'s comment says it also lists units equal to a former template value.

`TestFormerTemplateValues_DifferFromTheTemplate` pins the acceptance criterion on the table: for every entry, the module is registered, its template holds the unit at the same path, and the unit's former value differs from the current template's, compared as YAML nodes with the module's open maps; a row whose key the template lacks or whose value equals the template fails naming the module and unit.
`TestReconcileAll_MigratesToSparse` gains rows: an unmarked worktree `loom.yaml` holding `discussion_lenses: ""` and `review_max_bounces: 99` over a repo default that sets `discussion_lenses: tuned-fan` loses `discussion_lenses` and reports it stripped, so the worktree now resolves the default's value; a whole `claude.prompt_cache_ttl_roles` map equal to the former map is stripped; and a marked worktree keeps a former value byte for byte.

**ImpactSummary:** one new table and helper in `configsync`; the unmarked-worktree strip of `ReconcileAll` drops one more class of unit, reported through the existing `Stripped` list, so the walk and pair calls change with no caller edit.
