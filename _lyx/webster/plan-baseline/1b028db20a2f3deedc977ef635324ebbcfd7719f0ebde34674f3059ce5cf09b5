# Card 14 — configreg run settings

**Create:**
- `plan:internal/configreg#RunSettingsError` -> `type RunSettingsError struct`
- `plan:internal/configreg#RunSettingsError.Error` -> `func (e *RunSettingsError) Error() string`
- `plan:internal/configreg#RunSettingsError.Unwrap` -> `func (e *RunSettingsError) Unwrap() error`
- `plan:internal/configreg#PrepareRunSettings` -> `func PrepareRunSettings(settings map[string]any) (map[string][]byte, error)`
- `plan:internal/configreg#TestPrepareRunSettings` -> `func TestPrepareRunSettings(t *testing.T)`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/configreg#Module`
- `internal/configreg#Lookup`
- `internal/configengine#ValidateLocal`

**Intent:**
Add the one migrate-then-validate step D8's board write and D9's Seed-Child share, so the two can never judge a task's run settings differently.
`PrepareRunSettings` takes a task's `run_settings` as decoded from JSON, an object of module name to a nested object of that module's keys (dotted keys are not used), and returns each named module's migrated local-layer YAML document, in a new file beside `configreg.go`.
For each module, in name order: the name must be registered (`Lookup`); its value must be an object; an empty object drops the module; the object is marshalled to YAML, so a JSON whole number becomes a YAML integer; the module's `Migrate` hook, when non-nil, runs over it; and the result must pass `configengine.ValidateLocal(m.Spec(), doc)`, which checks that each key path exists in the template, sits under an open map or under any `models` alias, matches the template's shape where the template holds it, and is open to the local layer under D4.
Value semantics, such as a positive integer, are left to the module's load.

Every refusal is a `*RunSettingsError` carrying `Module` and the wrapped cause, whose text names the module and, for a key-level refusal, the key `ValidateLocal` named; it unwraps to the cause, so a key refusal still matches `configengine.ErrInvalid`.
A nil or empty `settings` returns an empty map and no error.

`TestPrepareRunSettings` is table-driven over the real registry: `{"loom": {"review_max_bounces": 2}}` yields a loom document holding exactly that integer key; a below-floor `step_idle_timeout_min` comes back raised by loom's `Migrate`; a `models` alias the template lacks and `opus.defaults.version` pass; `shuttle.claude_prompt_cache_ttl_roles.<role>` passes; an unknown module, a non-object module value, a key the template lacks outside any open map, a mapping where the template holds a scalar, and a D4-closed key (`darn.review_max_bounces`, `loom.parent_review_wait_min`) are each refused naming the module and the key; an empty module object is dropped.
The budget row for `internal/configreg` rises by one.

**ImpactSummary:** additive: one exported function and error type in configreg; only the budget row changes.
