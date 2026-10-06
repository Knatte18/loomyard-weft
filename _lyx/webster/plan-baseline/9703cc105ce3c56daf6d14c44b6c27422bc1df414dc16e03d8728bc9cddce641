# Card 7 — yaml-config-prune

**Edit:**
- `internal/yamlengine#`
- `internal/configengine#`
- `internal/configreg#`
- `internal/configsync#`
- `internal/configcli#`
- `internal/envsource#`

**Intent:**
Run the prune procedure (Shared Decisions, `prune-procedure`) on the test files of the YAML and config packages.

`internal/yamlengine` and `internal/configengine` carry large candidate sets of per-input parse and validation checks, the main fold leads.
`PATTERN-config-strictness` is pinned by `cmd/lyx`'s guard, not here, but a test that pins the strict `Load` refusal text or the `LoadOrTemplate` fallback is a unique assertion under Decision 2.
`internal/configcli`'s `TestBareConfigListsModulesAndVerbs` is a `failed` entry (Decision 8).
Its two `integration` files (`configcli_integration_test.go`, `reconcile_integration_test.go`) hold the tier-move and scenario-merge leads of this group.

**ImpactSummary:** Test files of six config and YAML packages only; no production change.

**Commit:** 7: Prune YAML and config tests
