# Card 9 — loom layered load

**Create:**
- `plan:internal/loomengine#ConfigSpec` -> `func ConfigSpec() configengine.Spec`

**Edit:**
- `internal/loomengine#LoadConfig` -> `func LoadConfig(dirs configengine.Dirs, module string) (Config, error)`
- `internal/loomengine#validateFanKeys` -> `func validateFanKeys(dirs configengine.Dirs, cfg Config) error`
- `internal/loomengine#DiscussionSpec` -> `func DiscussionSpec(layout *lyxcwd.Location, dirs configengine.Dirs, stencilsDir, parentName string, cfg Config, reg modelspec.Registry, slug string, autonomous bool) (shuttleengine.Spec, error)`
- `internal/loomengine#twillBlocks` -> `func twillBlocks(layout *lyxcwd.Location, dirs configengine.Dirs, stencilsDir string) (string, string, error)`
- `internal/loomengine#TestLoadConfig_Loads`
- `internal/loomengine#TestLoadConfig_NotInitialized`
- `internal/loomengine#TestLoadConfig_Refuses`
- `internal/loomengine#TestDiscussionSpec`
- `internal/loomengine#TestDiscussionSpec_PatternDirective`
- `internal/loomengine#TestDiscussionSpec_ReadsStencilAtCallTime`
- `internal/loomengine#TestDiscussionSpec_Refuses`
- `internal/loomengine#TestDiscussionSpec_Twill`
- `internal/loomengine#TestProducerSpecs_SkillsAndParentDirective`
- `internal/loomcli/arm.go`
- `internal/loomcli/wiring.go`
- `internal/webstercli/wiring.go`

**Uses:**
- `plan:internal/configengine#Spec`
- `plan:internal/configengine#Dirs`
- `plan:internal/fabricengine#ConfigDirs`

**Intent:**
Move loom onto the hub stack: `LoadConfig` takes `configengine.Dirs` and loads strictly through `configengine.Load(dirs, ConfigSpec())`, so in a hub its base layer is the repo default (D5) and a sparse worktree file and a local override sit above it.
`ConfigSpec` returns loom's `Spec` with `ConfigOpenMaps()` and `Categories` exactly as the overview's "Per-key classification" table lists loom's keys: `parent_review_wait_min`, `discussion_producer`, `discussion_fan` and `plan_fan` guard, every other key none, `review_max_bounces` included.

The burler and models reads loom makes are told the same dirs instead of card 8's transitional anchor-only geometry: `LoadConfig` passes its `dirs` to `modelspec.LoadRegistry` and to `validateFanKeys`, which passes them to `burlerengine.LoadConfig`.
Discussion-Write's twill lenses come from the same stack: `DiscussionSpec` and `twillBlocks` take the dirs, so a `twill-discussion` fan the prime defines in the repo default reaches the writer after migration; loomengine stays told (`PATTERN-told-geometry`) and never derives the board dir from the layout.
`internal/loomcli/wiring.go` and `arm.go` pass `fabricengine.ConfigDirs(location)` to `LoadConfig` and `DiscussionSpec`; webstercli passes it to `LoadConfig`; tests pass `configengine.Dirs{Anchor: dir}`.

`TestLoadConfig_Loads` gains hub-stack rows over a temp board dir and anchor: `review_max_bounces: 2` in the local file wins over the default's 3, a `review` list in the worktree file replaces the default's whole, and a strict load with the repo default absent errors naming its path; `TestLoadConfig_Refuses` gains a local `parent_review_wait_min`, `discussion_producer` and `plan_fan` each failing with `ErrInvalid` naming the local file and the key; `TestDiscussionSpec_Twill` gains a row whose `twill-discussion` fan lives only in the repo default.

**ImpactSummary:** re-signs `loomengine.LoadConfig` and `DiscussionSpec`: loomcli and webstercli pass told dirs, and loom's burler and models reads follow the same stack.
