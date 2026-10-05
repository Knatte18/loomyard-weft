# Card 2 — configengine threads open maps through load and set

**Edit:**
- `internal/configengine#Load`
- `internal/configengine#LoadOrTemplate`
- `internal/configengine#load`
- `internal/configengine#Set`
- `internal/configengine/config_test.go`
- `internal/configengine/set_test.go`

**Create:**
- `internal/configengine/doc.go`

**Uses:**
- `internal/yamlengine#FillMissing`
- `internal/yamlengine#MissingKeys`
- `internal/yamlengine#SetValues`

**Intent:**
`Load`, `LoadOrTemplate` and `Set` each gain a trailing variadic `openMaps ...string`, passed unchanged to `yamlengine.FillMissing`, `yamlengine.MissingKeys` and `yamlengine.SetValues`, so a module's open-map declaration reaches every consumer of its template (D3).
The strict/degrading choice is untouched (`PATTERN-config-strictness`): callers keep calling `configengine.Load(` or `configengine.LoadOrTemplate(` exactly as before, so `cmd/lyx/configstrictness_test.go`'s pinned sets do not move.

Bound: the parameter changes behaviour only for the paths a module declares, by card 1's rule; with none declared, every caller behaves as today.
What it lets through is card 1's: a present open map's value is no longer inspected by `Load`, and a mistyped key under it creates a new one in `Set`.

Tests, untagged: `Load` with an open map declared accepts a file whose map at that path holds keys the template lacks and lacks keys the template holds, and a file holding a list there; without the declaration the same file fails as today.
`Set` with `labels` declared adds `labels.x` to a map-shaped file and refuses it on a list-shaped one; `Set` on an undeclared nonexistent key still refuses with `unknown config key(s)`.

`doc.go` gains the package doc for `configengine`: the two loading policies, the in-memory fill, `Set`, and that a module's open maps are passed through to yamlengine.

**ImpactSummary:** three exported loaders gain an optional parameter; every current caller compiles unchanged.

**Commit:** 2: configengine threads open maps through load and set
