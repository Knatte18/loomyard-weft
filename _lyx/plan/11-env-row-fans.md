# Card 11 — env-row-fans

**Edit:**
- `internal/shedrecipe#Env`
- `internal/shedrecipe#burlerRoundEntry`
- `internal/shedrecipe#bouncerEntry`
- `internal/shedrecipe/entries_burler_test.go`
- `internal/shedrecipe#TestBouncerEntry_ClusterExcludes`

**Intent:**
The shed half of decision D1: a review segment's fan arrives as runtime config, threaded to both of its rows by row name, the way `RowReviewModels` threads models.
`Env` gains `RowClusterFans map[string]string`, keyed by row name; a row with no entry, and a nil map, take nothing from it.
`burlerRoundEntry` sets the profile's cluster fan from the row's entry.
A row that receives a non-empty fan from `RowClusterFans` while its recipe profile also sets a literal `cluster-fan` fails to build, with an error naming the row, both values and both sources (the recipe's `cluster-fan` and loom.yaml through the wiring) and the way forward of removing the recipe's `cluster-fan`.
`bouncerEntry` tells the Bouncer `ClusterExcludes` when the row's entry is non-empty, or when its own `cluster_excludes` key is true, so the judge is asked for `exclude_lenses` exactly when its round has a fan.

Bound: the map only turns a fan on for the rows the wiring names; the recipe stays the one place a row's engine and gates are set.

`entries_burler_test.go` gains a test that an env fan reaches the profile, that a row absent from the map keeps the recipe's value, and that an env fan beside a literal `cluster-fan` fails naming both.
`TestBouncerEntry_ClusterExcludes` gains rows for an env fan turning `ClusterExcludes` on and an empty entry leaving it off.

**ImpactSummary:** Two entry constructors read one new optional Env map; with the map nil every row builds exactly as before.
