# Card 1 — loom-config-keys

**Edit:**
- `internal/loomengine#Config`
- `internal/loomengine#LoadConfig`
- `internal/loomengine/template.yaml`
- `internal/loomengine#TestLoadConfig_Loads`
- `internal/loomengine#TestLoadConfig_Refuses`

**Prosa:**
- `docs/overview.md`
- `contracts/specs/llm-model-spec.md`

**Intent:** Add the eight new `loom.yaml` keys of D1, D2 and D6 to `Config`, each a plain `string` with its yaml tag: `seed`, `discussion_judge`, `plan_judge`, `webster_judge`, `discussion_seed`, `plan_seed`, `webster_seed` and `rework`.
Add each to `template.yaml` with an empty default and a comment stating its fallback: a per-segment judge key empty takes `judge`; a per-segment seed key empty takes `seed`, and `seed` empty takes that segment's resolved judge; `rework` empty takes `plan`.
Reword the `judge` comment so it names the per-segment keys that override it.
`LoadConfig` validates each new key through `modelspec.Parse` only when non-empty, naming the key in the error exactly as the `friction` and `driver` checks do, so a present-but-empty value loads cleanly and an ungrammatical one fails at load.
Update the file header comment of `config.go`, which lists the validated role keys, to name the new keys and their empty-means-fallback rule.
`TestLoadConfig_Loads` gains assertions that the template loads every new key empty and that a file setting them loads the values; `TestLoadConfig_Refuses` gains one row per new key with an ungrammatical value, asserting the error names that key.
`TestConfigTemplate_ContainsEveryConfigYAMLTag` already pins the template against the struct and needs no edit.
In `docs/overview.md`, extend the sentence that lists `loom.yaml`'s keys with the new ones.
In `contracts/specs/llm-model-spec.md`, extend the "Roles that use this notation" section so the Bouncer judge and seed passes and PR-Rework are named as roles with keys of their own, with the same fallback statements as the template comments.
No key changes the model any role runs on while every new key is empty.

**ImpactSummary:** Adds eight string fields and their load-time grammar checks to loomengine's Config; no reader of the new fields exists yet, and every existing key keeps its meaning.
