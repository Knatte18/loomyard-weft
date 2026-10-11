# Card 5 — run reads each run's archived shed status

**Create:**
- `tools/tokencount#resolveHub`
- `tools/tokencount/time_integration_test.go`

**Edit:**
- `tools/tokencount#run`
- `tools/tokencount/main_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `tools/tokencount#finishedRuns`
- `tools/tokencount#readShedRecord`
- `tools/tokencount#RunTally.useShedRecord`
- `tools/tokencount#repositoryFlag`
- `internal/fabricengine#RecordsWorktree`
- `internal/gitrepo#New`

**Intent:**
Wire the archived shed status into every counted run and retitle the report (decision record, Decisions 8 and 10).
Below, "the weft flag" is the flag `repositoryFlag` names.

`resolveHub` makes the hub flag's value absolute and clean: empty is `filepath.Dir(wd)`, a relative path joins onto `wd` (never `filepath.Abs`, which reads the process cwd outside `internal/lyxcwd`), then `filepath.Clean`.
It takes `filepath.Dir(wd)` as the current worktree's parent, assuming the tool runs from the worktree root, as the hub flag's existing default does and as the launchers ensure by changing to the repository root; run from a subdirectory, the correct hub is refused and the weft flag is the way through.
When the result differs from `filepath.Clean(filepath.Dir(wd))` and `weft` is empty it refuses with an error naming the given hub, the current directory's parent as the hub it assumes and the weft flag, saying the default weft would read another hub's records and that the weft flag must name that hub's weft repository; the flag is spelled through `repositoryFlag`.
The refusal is reached by the hub flag naming any directory other than the current worktree's parent with the weft flag empty; the same hub with a trailing slash or as a relative path resolves equal and passes.
It bounds Decision 8's wider weft default: the default can no longer pair one hub's slugs with another hub's records.

`run` calls `resolveHub` right after `lyxcwd.Getwd`, before anything is read.
The weft default, `fabricengine.RecordsWorktree` of the resolved location, now applies whenever the weft flag is empty, slugs named or not; outside a resolvable worktree `lyxcwd.Resolve` fails there, as Decision 8 accepts.
`run` calls `finishedRuns` once, for every invocation, and passes its map to `recentRuns` when no slugs are named.
After `CountRun` for each slug, a slug whose worktree `<hub>/<slug>` exists is in flight, since teardown removes it, and reads no archive tag, keeping the transcript-window span and the no-status line; any other slug's `readShedRecord` over `gitrepo.New` of the weft and the slug's tags, when it returns a record, goes to `useShedRecord`.
An `os.Stat` error other than not-exist aborts.
The check runs for every slug, named or picked by `recentRuns`, since Decision 8's reason, that teardown removes the worktree, holds for both.
The calibration reuses the same `gitrepo.Repo` for the weft.
The report title becomes `# Token usage and time by role`; the `.scratch/token-usage-by-role.md` path in the launchers stays.

Tests:
- a new table test `TestResolveHub` in `main_test.go` over `resolveHub` with a fixed `wd`: an empty hub resolves to the parent; the parent with a trailing slash and `..` resolve to the parent with no error; another directory with `weft` empty is refused, the error naming the weft flag; another directory with `weft` set resolves to it.
- `TestRunRefusesTheRetiredHistoryFlag` becomes the table test `TestRunRefuses`, keeping the retired-flag row and adding a row whose hub flag is a temporary directory and whose weft flag is empty: `run` returns the refusal naming the weft flag and writes no report.
- a new integration test `TestRunReadsArchivedShedStatus` in `time_integration_test.go`: a temporary git repository standing in for the weft, with lightweight `archive/<slug>/<sha>` tags on commits holding each run's `seed.json` and `status.json` at the paths `TreeRunFileRels` returns, committed with explicit `GIT_COMMITTER_DATE` values as `internal/gitrepo/blobread_integration_test.go` does, a temporary projects directory with timestamped transcripts, and a temporary hub in which one slug's worktree directory exists.
  `run`, given the hub, projects, weft and out flags and both slugs, writes a report titled `# Token usage and time by role` whose finished run carries the Producers table read from its newest tag, and whose in-flight run, which also has an older archive tag, prints the no-status line.

Raise `tools/tokencount`'s row in `cmd/lyx/testdata/test-budget.yaml` by the two new top-level tests.

**ImpactSummary:** `run` now always reads the weft's archive tags and each finished run's status, refuses a foreign hub without the weft flag, and retitles the report; with slugs named and the weft flag empty it now resolves the weft sibling, so it fails outside a resolvable worktree where it ran before.
