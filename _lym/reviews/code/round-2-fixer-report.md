# Round 2 fixer-report

## Changes

- **F1** (LOW, fixed as proposed), commit `4f995b2f`: `Prepare` in `internal/shuttleengine/codexengine/codexengine.go` resolves the told `Dirs.Records` with `filepath.EvalSymlinks` and uses the result as the writable root; on a resolve failure it logs a Warn through the engine's `warn` and keeps the told path.
  `TestPrepare_Launch` in `prepare_test.go` gained the row `RecordsToldThroughLink`, whose told `Records` is a symlink to the real checkout and whose want is the resolved path.
- **F2** (LOW, fixed as proposed), commit `84911457`: `ConfigDirs` in `internal/fabricengine/junctionnames.go` treats the weft stat as the Bolt stat is treated, through a new helper `existingDirOrLog`: a directory sets the field, absence stays silent, and any other stat error logs a Warn naming the path and error.
  The doc comment's last sentence now says the logging covers either checkout.
- **Self-check findings** (both LOW, fixed as proposed), commit `a8bf9a5c`: the `Prepare` doc comment and `doc.go` now state the writable root's symlink resolution and its fallback; the `ConfigDirs` doc comment drops its mention of a launch consumer.

The gate `lym gate test ./internal/shuttleengine/codexengine ./internal/fabricengine` passed at tier 0 after F1 and F2; the doc-only commit was checked with `go vet` on both packages.
No test covers the weft stat-failure branch of `ConfigDirs` or the resolve-failure fallback in `Prepare`: neither finding asked for one, and a non-absence stat error is hard to build with the fixtures.

## Self-check

- fix-code-correctness: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.
- fix-consistency, LOW: the `Prepare` doc and `doc.go` omit the writable root's symlink resolution — fixed as proposed.
- fix-consistency, LOW: the `ConfigDirs` doc names a launch consumer — fixed as proposed.

Every fork returned a report.

## Disputed

None.

## Deferred

None.
