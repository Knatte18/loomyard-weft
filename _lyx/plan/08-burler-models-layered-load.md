# Card 8 — burler and models layered load

**Create:**
- `plan:internal/burlerengine#ConfigSpec` -> `func ConfigSpec() configengine.Spec`
- `plan:internal/modelspec#ConfigSpec` -> `func ConfigSpec() configengine.Spec`

**Edit:**
- `internal/burlerengine#LoadConfig` -> `func LoadConfig(dirs configengine.Dirs) (Config, error)`
- `internal/modelspec#LoadRegistry` -> `func LoadRegistry(dirs configengine.Dirs) (Registry, error)`
- `internal/burlerengine#loadSeededConfig`
- `internal/burlerengine#TestLoadConfig`
- `internal/modelspec#TestLoadRegistry_FileExtends`
- `internal/modelspec#TestLoadRegistry_FileOverridesWholeEntry`
- `internal/modelspec#TestLoadRegistry_RejectsInvalidEntries`
- `internal/modelspec#TestLoadRegistry_YieldsBuiltins`
- `internal/modelspec#TestConfigTemplate_LoadsWithLiveEntries`
- `internal/modelspec/modelspec.go`
- `internal/loomengine#LoadConfig`
- `internal/loomengine#validateFanKeys`
- `internal/loomengine#twillBlocks`
- `internal/loomengine#discussionTableRegistry`
- `internal/loomengine#TestDiscussionSpec`
- `internal/loomengine#TestDiscussionSpec_PatternDirective`
- `internal/loomengine#TestDiscussionSpec_ReadsStencilAtCallTime`
- `internal/loomengine#TestDiscussionSpec_Refuses`
- `internal/loomengine#TestDiscussionSpec_Twill`
- `internal/loomengine#TestProducerSpecs_SkillsAndParentDirective`
- `internal/loomengine#TestResolveDriver`
- `internal/loomengine#TestResolveDriver_EmptyDriverIsOffDefault`
- `internal/loomengine#TestResolveDriver_Refuses`
- `internal/loomengine#TestPlanSpec`
- `internal/loomengine#TestPlanSpec_PatternDirective`
- `internal/loomengine#TestPlanSpec_Refuses`
- `internal/loomengine#TestResolveJudge`
- `internal/loomengine#TestResolveReview`
- `internal/loomengine#TestResolveReviewAndJudge_MalformedSpec`
- `internal/loomengine#TestReworkSpec`
- `internal/loomengine#TestReworkSpec_MissingStencil`
- `internal/darnengine#newTestRegistry`
- `internal/loomcli#TestWire_DefaultConfig`
- `internal/burlercli/wiring.go`
- `internal/loomcli/wiring.go`
- `internal/webstercli/wiring.go`
- `cmd/lyx/configstrictness_test.go`

**Uses:**
- `plan:internal/configengine#Spec`
- `plan:internal/configengine#Dirs`
- `plan:internal/configengine#Layers`
- `plan:internal/fabricengine#ConfigDirs`

**Intent:**
Move the two seed-only modules onto the layer stack while keeping their own readers, since neither goes through `Load`: each reads its layers through `configengine.Layers(dirs, ConfigSpec())` and no longer forms a config path itself.

- `burlerengine.ConfigSpec` returns burler's `Spec` with `Categories` as the overview's table lists: `lenses` and `fans` both guard, so a local `burler.yaml` can hold no key (D4).
  `LoadConfig` decodes each layer strictly as today, naming that layer's file in a parse error, and merges them in stack order per name, so a lens string or a fan list in an upper layer replaces the lower one whole; no layer present still returns the zero `Config`, and `ResolveFan` keeps falling back per name to the template.
- `modelspec.ConfigSpec` returns models's `Spec` with `WholeEntries: true` and `Categories` listing each template alias as none.
  `LoadRegistry` decodes and validates each layer's entries as today, naming that layer's file, then lays them over `builtins()` in stack order by whole-entry replacement, so an upper layer's alias replaces the lower one outright and inherits none of its `defaults`; no layer present still yields the built-ins.
  `modelspec` keeps its leaf import set (`PATTERN-leaf-packages`): the standard library, `configengine` and `yaml.v3`.

The production callers in burlercli, loomcli and webstercli pass `fabricengine.ConfigDirs(location)` or, standalone, `configengine.Dirs{Anchor: stateDir}`.
Inside loomengine, `LoadConfig`, `validateFanKeys` and `twillBlocks` pass the transitional `configengine.Dirs{Anchor: baseDir}` (the layout's anchor for twill) that keeps today's single-file behaviour until card 9 tells them the hub dirs; tests pass `configengine.Dirs{Anchor: dir}`.
The comment in `cmd/lyx/configstrictness_test.go` that says the two own-loader modules resolve their path with `configengine.ConfigFile` is corrected to say they read their layers through `configengine.Layers`, which the guard's substring scan still does not see; `internal/modelspec/modelspec.go`'s fallback note is updated the same way.

`TestLoadConfig` (burler) gains rows over a temp board dir and anchor: a fan defined only in the repo default resolves, a worktree-file lens replaces the default's whole, and a local `lenses` entry fails the load with `ErrInvalid` naming the local file.
`TestLoadRegistry_FileOverridesWholeEntry` gains the layered rows: a local alias without a `defaults` block resolves with no defaults from the layers below, a local alias the template lacks loads, and a local `opus.defaults.version` entry loads; `TestLoadRegistry_RejectsInvalidEntries` gains an invalid local entry refused naming the local file.

**ImpactSummary:** re-signs `burlerengine.LoadConfig` and `modelspec.LoadRegistry`: three CLI wirings, loomengine's internal loads and many loomengine spec tests pass dirs; outside a hub each reads the one file it read before.
