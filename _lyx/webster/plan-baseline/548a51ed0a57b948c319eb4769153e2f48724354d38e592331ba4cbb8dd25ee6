# Card 6 — The never-tracked mirror of a config dir

**Create:**
- `plan:internal/configengine#ScratchDir` -> `func ScratchDir(baseDir string) string`

**Edit:**
- `internal/configengine#StagingFile`
- `internal/configengine/config_test.go`

**Intent:**
Decision 2 places the build stamp and the hub lock at the `.lyx` mirror of the hub-wide config dir, and card 7's `hubreconcile` builds both paths from a base dir it is told; configengine is the package that already names `<base>/_lyx/config` (`ConfigDir`) and its `.lyx` counterpart for one file (`StagingFile`), so it names the mirrored dir too.

`ScratchDir(baseDir)` (in `config.go`, beside `ConfigDir`) returns `<baseDir>/.lyx/config`, joined from `lyxdirs.DotLyxDirName` and the package's `configDirName` (`PATTERN-lyxdirs-single-declarer`, `PATTERN-durable-vs-ephemeral-state`).
Its doc comment says it is the never-tracked mirror of `ConfigDir`, for files a caller keeps beside the config that must never be committed.
`StagingFile` joins its module file onto `ScratchDir(baseDir)` instead of repeating the two segments; its result is unchanged.

Tests: `config_test.go`'s `TestConfigPaths` table gains a `ScratchDir` row, `filepath.Join(base, lyxdirs.DotLyxDirName, "config")`, beside its `StagingFile` row, which keeps pinning `StagingFile`'s unchanged path.

**ImpactSummary:** One additive path accessor in configengine; `StagingFile` keeps its result, so its callers are unaffected.

**Commit:** 6: configengine: name the never-tracked mirror of a config dir
