```yaml
lens: code-affected-set
outcome: reported
```

**Lens code-affected-set: I found nothing the diff leaves wrong.**

Set found per change:

| Change | What I found | Covered |
|---|---|---|
| `SetValues` and `SetSparse` take a `shapes` argument | Two production callers, both updated: `internal/configengine/set.go:81` and `:166`. The test callers in `yamlengine/set_test.go` and `layers_test.go` are updated too. A repo-wide grep found no other caller. | yes |
| New `Spec.OpenMapShapes` field | Only the gate spec sets it (`internal/gateslot/config.go:109`). `configengine/set_test.go:397` and `configreg_test.go` build `Spec` literals without it, which is fine because the field defaults to nil. `configreg/configreg.go:221` fingerprints `OpenMaps` only, so shapes need no entry there. | yes |
| `--set` help text (`configcli.go:481-482`) | The example changed from `labels.quarry` to the gate one. I found no test asserting the old help text or a help size cap. The `labels.quarry` example survives in `configcli/doc.go:52`. | yes |
| Docs for the new `--set` behaviour | `yamlengine/doc.go`, `configengine/doc.go`, `configcli/doc.go:56-57`, `gateslot/doc.go:59`, `gateslot/config.go:103` and `template.yaml:10` all describe the nested form. Nothing else in the tree describes `--set` on open maps: no stencils, specs or other docs. | yes |
| Other open-map modules (`board`, `darn`, `drift`, `landing`, `loom`, `tapestry`, `reed`, `claude`) | None declares a shape, so they keep the dotted-rest-as-one-name behaviour, and the unshaped case has a `TestSetValues` row. | yes |
| PATTERN entries | `PATTERN-config-layers` is untouched, and nothing in it conflicts with the change (`configengine` is still the sole writer, and the layer rules are unchanged). The directive declares no PATTERN changes. | yes |
| Commit messages and departures | There is one commit and it carries no `Pattern-Departure:` trailer. None is needed. | yes |

Tool calls:
1. grep for the `labels.quarry` and `LYM_TEST_TIER` mentions across the repo: to find docs, fixtures and tests that mention the old or new example. My context lacked this.
2. grep of tests for help-size assertions and `Spec` literals: to check for tests that assert on the changed help text or `Spec` shape. My context lacked this.
