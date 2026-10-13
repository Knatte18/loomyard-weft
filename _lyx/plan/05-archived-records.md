# Card 5 — Read each run's archived seed and status from the weft

**Create:**
- `plan:tools/tokencount#ArchiveTag` -> `type ArchiveTag struct`
- `plan:tools/tokencount#ArchiveReader` -> `type ArchiveReader interface`
- `plan:tools/tokencount#readArchivedRun` -> `func readArchivedRun(records ArchiveReader, slug string, tags []ArchiveTag) (*ArchivedRun, error)`
- `plan:tools/tokencount#TestReadArchivedRun` -> `func TestReadArchivedRun(t *testing.T)`
- `plan:tools/tokencount#TestRunRefusesAForeignHub` -> `func TestRunRefusesAForeignHub(t *testing.T)`
- `tools/tokencount/archive_integration_test.go`

**Edit:**
- `tools/tokencount#finishedRuns` -> `func finishedRuns(weft string) (map[string][]ArchiveTag, error)`
- `tools/tokencount#archivedSlugs` -> `func archivedSlugs(refs string) (map[string][]ArchiveTag, error)`
- `tools/tokencount#TestArchivedSlugs`
- `tools/tokencount/main.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Decisions 3, 8 and 9: read each counted run's archived `seed.json` and `status.json` from its archive tag in the weft, and feed them to `MeasureRunTime`.

`ArchiveTag` holds `Name` (`archive/<slug>/<sha>`), `SHA` (the full commit object name) and `Committed time.Time` (the commit's committer date).
`finishedRuns` lists the tags with `gitexec.Run` (`PATTERN-gitexec-checked-call`), `for-each-ref --format=%(refname) %(objectname) %(committerdate:iso-strict) refs/tags/archive/`; archive tags are lightweight tags on weft commits, so both fields name the commit.
`archivedSlugs` parses that output into tags per slug, each slug's tags newest first by `Committed`; a line whose refname is not `refs/tags/archive/<slug>/<sha>` is skipped as now, and a matching line whose date does not parse returns an error.
`ArchiveReader` is the one method `FileAtRevision(rev, relPath string) ([]byte, error)`; `*gitrepo.Repo` and the test package's `fakeHistory` both satisfy it.

`readArchivedRun` takes the slug's tags, newest first, and returns nil with no error when there are none.
It reads at `tags[0].SHA`, the newest tagged commit (Decision 8), with `Floor` set to `tags[1].Committed` when a second tag exists.
The paths come from `shedrun.RunRecordRels(slug)` (card 1); a slug that fails `ValidateRunID` returns an error naming the slug, the tag name and both files, `seed.json` and `status.json` (Decision 9).
A real input reaches it: the slug `.` passes `CountRun`, since `filepath.Join(*hub, ".")` cleans to the hub and its project directory can hold sessions, and `ValidateRunID(".")` refuses it.
A `status.json` absent at the revision (`gitrepo.ErrPathNotAtRevision`) returns nil, the no-status fallback; an absent `seed.json` leaves the seed empty, so the span starts at the earliest transcript line.
A seed that `shedrun.DecodeSeed` refuses, or a status that does not decode as `shedengine.Status`, returns an error naming the slug, the tag name and the file (`seed.json` or `status.json`).

In `main.go`'s `run`:
- The weft default, `fabricengine.RecordsWorktree` of the resolved worktree, now applies whenever -weft is empty, named slugs included.
  This lets a slug be read against the current hub's weft while `-hub` names another hub; the bound is a refusal: when -weft is empty and `-hub`, joined onto `wd` (from `lyxcwd.Getwd`, which `run` already holds) when relative and then cleaned with `filepath.Clean`, differs from `filepath.Dir(wd)` cleaned the same way, `run` returns an error naming -weft before it resolves the worktree or reads anything.
  `filepath.Abs` is not used, since it queries the cwd itself (`PATTERN-cwd-resolution`).
  Outside a resolvable worktree with -weft empty, `run` fails on the resolve error, named slugs included.
- The archive tags are read once with `finishedRuns(*weft)`; with no slugs named, `recentRuns` gets the set of their slugs as before.
- For each slug, after `CountRun`: when `<hub>/<slug>` still exists, the run is in flight, since teardown removes the worktree, so no tag is read and `archived` is nil, even when an older run's tag for that slug exists; this bounds the in-flight slug that would otherwise take an older run's archive tag.
  Otherwise `readArchivedRun(gitrepo.New(*weft), slug, tags[slug])`.
  `MeasureRunTime(run, archived)`'s result goes into `run.Time`; any error aborts the report.
- The package doc in `main.go` replaces its sentence saying the -weft default "applies whenever the flag is empty and either no slugs are named or the calibration is asked for" and states the new weft default, the `-hub` refusal and its bound, the in-flight rule, and which tag is read; the -weft flag's usage string drops nothing it says now.

Tests:
- `TestArchivedSlugs` takes refs with sha and date fields: several tags of one slug come back newest first by committer date, a malformed refname is skipped, an unparseable date is an error.
- New `TestReadArchivedRun`, table-driven over `fakeHistory` (files keyed `<sha>:<path>`, paths built from `shedrun.RunRecordRels`): two tags read the newest one's status and set `Floor` to the older one's date; no tags returns nil; a status absent at the revision returns nil; a seed absent keeps an empty seed; a seed with an unknown key and a status that is not JSON each return an error whose text holds the slug, the tag and the file name; the slug `.` with one tag returns an error whose text holds `.`, the tag, `seed.json` and `status.json`.
- New `TestRunRefusesAForeignHub` in `main_test.go`, not parallel because it calls `t.Chdir` into a temp directory: `-hub` naming another temp directory with -weft empty and a slug returns an error naming -weft and writes no report; `-hub` naming the temp directory's parent with a trailing separator, and as the relative `..`, each return an error that is not the -weft refusal (the resolve error, the temp directory being no worktree).
- New `archive_integration_test.go` (`//go:build integration`) with `TestRunReadsArchivedStatus`, not parallel because it calls `t.Chdir`: a hermetic git fixture (`gitkit.MustRun`, committer dates set through `GIT_COMMITTER_DATE` with `t.Setenv`) commits slug `alpha`'s `seed.json` and `status.json` twice with different histories and tags each commit `archive/alpha/<sha>`, and tags a commit for slug `live` whose `seed.json` and `status.json` are valid, the status holding a history entry, so only the in-flight rule keeps `live` from a Producers table; `run` with -weft the fixture, `-hub` a temp hub holding a `live` directory and `-projects` holding sessions for both slugs writes a report titled `# Token usage and time by role` whose `alpha` Producers table comes from the newer tag's history, and whose `live` section carries the no-status line and no Producers table.
The decode-failure abort is pinned at the unit tier by `TestReadArchivedRun`, so the integration test carries no failing commit.
The `tools/tokencount` row of `cmd/lyx/testdata/test-budget.yaml` rises by three.

**ImpactSummary:** `finishedRuns` and `archivedSlugs` change shape with one caller each, `run` and `finishedRuns`, both on this card; `run` now reads the weft for named slugs and refuses a foreign `-hub` without -weft, which the calibrate integration test avoids by naming -weft.

**Commit:** 5: tokencount: read each run's archived seed and status from the weft
