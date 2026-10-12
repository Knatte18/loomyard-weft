# Card 6 — run-wiring

**Edit:**
- `tools/tokencount#run`
- `cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `plan:tools/tokencount#TestRunRefusesAForeignHubWithoutWeft` -> `func TestRunRefusesAForeignHubWithoutWeft(t *testing.T)`
- `tools/tokencount/run_integration_test.go`

**Uses:**
- `plan:tools/tokencount#readArchivedRun`
- `plan:tools/tokencount#RunTally.resolveTiming`
- `tools/tokencount#finishedRuns`
- `tools/tokencount#repositoryFlag`
- `internal/fabricengine#RecordsWorktree`
- `internal/lyxcwd#Resolve`

**Intent:** Wire the status read and the timing into `run`, Decisions 3, 8 and 10.
The repository default, `fabricengine.RecordsWorktree` of `lyxcwd.Resolve(wd)`, now applies whenever the `weft` flag, `repositoryFlag`, is empty, not only with no slugs or with `-calibrate`; outside a resolvable worktree a run with that flag empty fails with `Resolve`'s error, named slugs included.
This lets a run with named slugs read the current hub's weft against another hub's sessions when `-hub` names another hub.
Bound: when the `weft` flag is empty and `-hub`, after defaulting, differs from the parent of `wd`, both compared as absolute cleaned paths through `filepath.Abs` and `filepath.Clean`, `run` returns an error that names the `weft` flag as the way forward, before any read.
`-hub` spelled with a trailing slash or as a relative path of the same hub is not refused.
`finishedRuns` is read once for every run, since its tags now feed the status read as well as the no-slug listing.
For each counted slug, named or listed: when the directory `<hub>/<slug>` exists the run is in flight, since teardown removes the worktree, so no tag is read and `resolveTiming` gets nil, which keeps an in-flight run that reuses an older run's slug from taking the older archive; else `readArchivedRun` over `gitrepo.New(weft)` with the slug's tags, and `resolveTiming` with the result, an error aborting the report.
The report title becomes "Token usage and time by role"; the `-out` path stays.
`TestRunRefusesAForeignHubWithoutWeft`, in `main_test.go`, calls `run` with `-hub` set to a fresh temporary directory, the `weft` flag absent and one slug, and asserts the error names the `weft` flag and nothing was written; this is the input that reaches the refusal, since flag parsing and `Getwd` pass before it.
`run_integration_test.go`, under `//go:build integration`, holds `TestRunReadsArchivedStatusFromTheWeft`, which cannot run in parallel because it changes the working directory, as its comment says: it builds a hub under `t.TempDir()` with a prime worktree, a `git init` repository, and the prime's weft sibling named by `fabricengine.RecordsWorktree`, another repository holding a commit with `_lyx/shed/alpha/seed.json` and `status.json` tagged `archive/alpha/<short sha>` as a lightweight tag and an older commit for the same slug tagged the same way, with transcripts for `alpha` and for a slug `beta` whose worktree directory exists under the hub, all through `gitkit`; it changes into the prime and calls `run` with `-hub` spelled with a trailing slash, `-projects` and the `weft` flag absent, naming both slugs, and asserts the report holds the new title, `alpha`'s Producers table with the newer status's producer, and `beta`'s no-status line.
Two new top-level tests, so the `tools/tokencount` row in `cmd/lyx/testdata/test-budget.yaml` rises by two.

**ImpactSummary:** One function changes, `run`; the retired-flag test and the calibration integration test keep passing since they pass the `weft` flag or fail at flag parsing.
