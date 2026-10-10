# Card 8 — darn-config

**Create:**
- `internal/darnengine/doc.go`
- `internal/darnengine/template.yaml`
- `plan:internal/darnengine#configTemplate` -> `var configTemplate string`
- `plan:internal/darnengine#ConfigTemplate` -> `func ConfigTemplate() string`
- `plan:internal/darnengine#Config` -> `type Config struct`
- `plan:internal/darnengine#LoadConfig` -> `func LoadConfig(baseDir, module string) (Config, error)`
- `plan:internal/darnengine#verifyWayForward` -> `const verifyWayForward = "run \"lyx config darn --set verify=<command>\" from the prime, then \"lyx loom resume\""`
- `plan:internal/darnengine#VerifyCommand` -> `func VerifyCommand(baseDir string) (string, error)`
- `internal/darnengine/config_test.go`

**Edit:**
- `internal/configreg#Modules`
- `internal/configreg/configreg_test.go`
- `cmd/lyx#configStrictnessStrictSet`
- `cmd/lyx/testdata/test-budget.yaml`
- `docs/overview.md`

**Intent:** D7: `darn.yaml` is a hub-wide config module owned by a new package, `internal/darnengine`, the engine that composes the darn writer's session and reads its verify command, so the registration oracle sees one `ConfigTemplate` per package as it does for landing and loom.
`Config` mirrors the template's four keys, `writer` (`opus[medium]`), `writer_timeout_min` (`180`), `verify` (`""`) and `verify_attempts` (`3`), each with the template comment stating its meaning and that `verify` runs any shell command in the task worktree, which is why the module is hub-wide.
`LoadConfig` loads strictly through `configengine.Load` (`PATTERN-config-strictness`), maps the not-initialized error the way landing's loader does, and validates the `writer` model-spec grammar at load time;
it leaves `verify` and `verify_attempts` to their readers.
`VerifyCommand` loads the file strictly at each call and returns the `verify` value, or an error, never `""`, when the file is missing or the value is empty, its message naming `verifyWayForward`, so the darn gate, Publish and Finalize each stop at call time instead of skipping their verify.
`Modules` registers `{Name: "darn", Template: darnengine.ConfigTemplate, HubWide: true}` in sort order, so the hub seeds and reconciles the file from the template, `lyx config darn --set` writes it from the prime, and the hub-wide write refusal covers it.
`darnengine` enters the config-strictness strict set.
Tests: `config_test.go` covers the template defaults, an invalid writer spec, the not-initialized mapping, and `VerifyCommand`'s value, missing-file and empty-value answers with the way forward in the message, as rows of a table test per function;
the configreg flags test adds `darn` to the hub-wide set.
The overview's directory tree gains the `internal/darnengine` row.
The sandbox coverage guard needs no change: `darn` registers no CLI module, and the core suite's config scenario already covers `lyx config <module>`.

**ImpactSummary:** One new leaf package and one registry row; every config surface renders the new module from the registry, and no existing module's loading changes.
