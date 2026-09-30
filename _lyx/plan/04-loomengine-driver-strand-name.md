# Card 4 — loomengine-driver-strand-name

**Create:**
- `plan:internal/loomengine#LoomDriverStrandName` -> `const LoomDriverStrandName = "loom-driver"`

**Edit:**
- `internal/loomcli#driverStrandDisplayName`

**Intent:**
Batten needs the loom driver strand's name to check its liveness from prime, but it lives in `internal/loomcli` as an unexported constant, and `battencli` must not import another CLI package.
Declare `LoomDriverStrandName` in `internal/loomengine/driver.go`, carrying over the existing doc comment's reason the name is stable (reed's add has no upsert semantics, so every add and every lookup must use it).
Redefine `driverStrandDisplayName` in `internal/loomcli/bootstrap.go` as `loomengine.LoomDriverStrandName`, with its doc comment pointing at the engine constant, so every existing loomcli use and `driverTeardownCommand` keep their spelling.

This is a pure relocation of a value: the existing bootstrap and driver-prompt tests cover it.

**ImpactSummary:** One constant's value moves to loomengine; loomcli's constant becomes an alias with the same string, so no behavior changes.
