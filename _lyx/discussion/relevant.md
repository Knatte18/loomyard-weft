# Relevant files

- `internal/lenses/config.go` `Config`, `ConfigSpec`, `Load`, `ResolveFan`, `CheckLoad` — the lens module's decode shape, spec and loaders the language and remove keys change
- `internal/lenses/template.yaml` — the base lens library whose Go-worded lenses move to the go layer
- `internal/lenses/doc.go` — module doc that states a template lens cannot be removed
- `internal/lenses/config_test.go` — pins seeded counts and the no-ignore scan the layer checks extend
- `internal/configengine/layers.go` `Spec`, `Layer`, `LayerFile` — the spec the overlay joins and the layer model
- `internal/configengine/config.go` `LoadOrTemplate`, `fallbackToTemplate` — load paths that fill and judge against the template
- `internal/configengine/set.go` — sparse set writes judged against the template
- `internal/configengine/doc.go` — layer and base-layer docs the overlay extends
- `internal/configsync/configsync.go` — reconcile, prune, strip and template-equal drop paths judging files against `m.Template()`
- `internal/configreg/configreg.go` `Modules` — registers the lenses module with its spec and check
- `internal/languageconfig/doc.go` — the language module that names no language and stays unchanged
- `internal/burlercli/wiring.go` — burler's lenses loads
- `internal/loomengine/twill.go` — twill writers' lenses load
