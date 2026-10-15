# Card 5 — shuttleengines-registry

**Create:**
- `internal/shuttleengines/doc.go`
- `plan:internal/shuttleengines#constructor` -> `type constructor func(cfg shuttleengine.Config) shuttleengine.Engine`
- `plan:internal/shuttleengines#registry` -> `var registry map[string]constructor`
- `plan:internal/shuttleengines#New` -> `func New(cfg shuttleengine.Config) (shuttleengine.EngineSet, error)`
- `plan:internal/shuttleengines#build` -> `func build(cfg shuttleengine.Config, constructors map[string]constructor) (shuttleengine.EngineSet, error)`
- `plan:internal/shuttleengines#Names` -> `func Names() []string`
- `plan:internal/shuttleengines#SessionNamer` -> `func SessionNamer() reedengine.SessionNamer`
- `internal/shuttleengines/shuttleengines_test.go`
- `internal/shuttleengines/seam_enforcement_test.go`

**Delete:**
- `internal/cliwire/sessionnamer.go`

**Edit:**
- `internal/reedcli#enterSession`
- `internal/cliwire#cliwireAllowedImports`
- `internal/cliwire/doc.go`
- `pattern/PATTERN-told-geometry.md`
- `internal/reedengine/doc.go`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** Add the engine registry, `internal/shuttleengines`, the one production package that imports a concrete engine (D1, D2).

`internal/shuttleengines/shuttleengines.go` holds `registry`, the registered engine names with their constructors: `"claude"` builds `claudeengine.NewFromConfig(cfg)`.
`New(cfg)` is `build(cfg, registry)`.
`build` constructs one engine per registered name from the same `cfg` and returns `shuttleengine.NewEngineSet(cfg.Engine, engines)`.
It refuses, before returning a set, a `cfg.Engine` that names no registered engine, whichever config layer set it: `shuttle: shuttle.yaml's engine: "<name>" names no registered engine; registered engines: <sorted names>; way forward: set engine: in shuttle.yaml to one of them`.
D8's refusal of an engine that declares forks without fork audit lands in card 11, since every `Engine` satisfies `ForkAuditor` until card 11 drops the audit methods from it.
`Names` returns the registry's names, sorted.
`SessionNamer` moves here from `cliwire` with its doc comment and keeps returning `claudeengine.New()` until `engine-config-seams` makes it per provider.
The package is told its `Config` and never imports `internal/lyxcwd` (`PATTERN-told-geometry`).

`doc.go` states the package: the registry of named engines, the only production importer of a concrete engine package, the set it builds from a shuttle config, and why `SessionNamer` lives here for now.

`internal/cliwire/sessionnamer.go` is deleted.
`reedcli`'s `enterSession` calls `shuttleengines.SessionNamer()` instead of `cliwire.SessionNamer()`.
`cliwireAllowedImports` drops `internal/reedengine` and `internal/shuttleengine/claudeengine`, which only `sessionnamer.go` imported; the scan's no-stale check then holds.
`cliwire/doc.go`'s fixed-dependency sentence drops the same two and the "The last two serve SessionNamer alone" sentence with its follow-on line.
`pattern/PATTERN-told-geometry.md` adds `shuttleengines` to its Bound packages list after `shuttleengine`.
`internal/reedengine/doc.go`'s name-repair sentence says the provider package implements the `SessionNamer` seam and `internal/shuttleengines` fills it, no longer `cliwire`.
`docs/overview.md`'s module tree gains `internal/shuttleengines/` in its sorted place, described as the engine registry and the only production importer of a concrete engine;
its shuttle row names `internal/shuttleengines` among its packages, and its "Claude is the only v1 engine" sentence says the engine is picked from the registry by `shuttle.yaml`'s `engine:` key, Claude being the one registered engine.

`shuttleengines_test.go` (package `shuttleengines`) adds:
- `TestNew`, table-driven: the template config builds a set whose default resolves to a `*claudeengine.Claude` and whose `Names` are `[claude]`; a `Config{Engine: "bogus"}` is refused with an error naming `bogus`, the registered engines and `engine:` in `shuttle.yaml`; a config loaded by `shuttleengine.LoadConfig` over a temp anchor whose local layer file, `configengine.LocalFile(anchor, "shuttle")`, holds `engine: bogus` is refused the same way.
- `TestNames_EqualModelspecEngineNames`: `Names()` equals `modelspec.EngineNames()` (D12).

`seam_enforcement_test.go` adds `TestToldGeometryInvariant_AllowlistOnly`, which runs `scankit.AssertImportAllowlistNoStale` over `internal/shuttleengines` with the allowlist `internal/shuttleengine`, `internal/shuttleengine/claudeengine` and `internal/reedengine`, the shape `internal/reportengine`'s seam test has.
Add the row `internal/shuttleengines: 3` to `test-budget.yaml`, in its sorted place after `internal/shuttleengine/claudeengine`.

**ImpactSummary:** New package; moves SessionNamer's one caller in reedcli, shrinks cliwire's allowlist and doc, rewords reed's and the overview's docs, and adds one PATTERN bound package and one budget row.
