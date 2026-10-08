# Card 3 — burler-exclude-held

**Edit:**
- `internal/burlerengine#Profile`
- `internal/burlerengine#Profile.validate`
- `internal/burlerengine#applyClusterExclude` -> `func applyClusterExclude(fan string, lenses []Lens, exclude, held []string) []Lens`
- `internal/burlerengine#TestProfileValidate_ClusterExclude`
- `internal/burlerengine/doc.go`

**Intent:**
The engine half of decision D5's empty-fan rule.
`Profile` gains `ClusterExcludeHeld []string`: the exclusions carried from earlier rounds of the generation, a subset of `ClusterExclude` that the caller fills.
`applyClusterExclude` drops `exclude` from the resolved fan as today, ignoring with a warning a name the fan lacks.
When that would leave no lens, it applies `held` instead and logs, as a warning, the names of `exclude` it dropped;
when `held` would empty the fan too, it runs the whole fan, as today.
`Profile.validate` passes `p.ClusterExcludeHeld` through, and treats `ClusterExcludeHeld` set with an empty `ClusterFan` as the same error `ClusterExclude` gets.
The survivors stay the single `clusterLenses` value the prompt and the fork audit's exact count read.

Bound: the fallback only ever runs more lenses than `exclude` asked for, never fewer, and never zero.

`TestProfileValidate_ClusterExclude` gains rows for a full exclusion that falls back to the held set, a held set that would also empty the fan running the whole fan, and a held set without a fan being a validate error.
The `burlerengine` package doc's `ClusterExclude` paragraph describes the held fallback.

**ImpactSummary:** One caller of applyClusterExclude (Profile.validate) changes with it; Profile gains a field the burler producer fills in card 8.
