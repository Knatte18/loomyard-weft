# Card 6 — wire the run and title

**Create:**
- `plan:tools/tokencount#checkHubMatchesWorktree` -> `func checkHubMatchesWorktree(hub, wd string) error`
- `plan:tools/tokencount#TestCheckHubMatchesWorktree` -> `func TestCheckHubMatchesWorktree(t *testing.T)`
- `tools/tokencount/shedstatus_integration_test.go`

**Edit:**
- `tools/tokencount#recentRuns` -> `func recentRuns(projects, hub, prime string, finished map[string][]ArchiveTag, last int) ([]string, error)`
- `tools/tokencount#run`
- `tools/tokencount#TestRecentRuns`
- `cmd/lyx/testdata/test-budget.yaml`

**Delete:**
- `tools/tokencount#finishedRuns`
- `tools/tokencount#archivedSlugs`
- `tools/tokencount#TestArchivedSlugs`

**Uses:**
- `plan:tools/tokencount#archiveTags`
- `plan:tools/tokencount#ReadShedRecord`
- `plan:tools/tokencount#RunTally.SetTime`
- `tools/tokencount#repositoryFlag`
- `tools/tokencount#TestCalibrateGitBackedMatchesInMemory`
- `plan:tools/tokencount#stampedLine`
- `tools/tokencount#writeLines`

**Intent:**
Decisions 8 and 10: `run` reads every counted run's archived status and sets its time, under the weft default and the hub guard Decision 8 bounds.
The repository flag (the `repositoryFlag` constant) is the one flag that names the weft repository; its default, the current worktree's weft sibling, now applies whenever the flag is empty, not only with no slugs or a calibration: `run` calls `checkHubMatchesWorktree(*hub, wd)`, then `lyxcwd.Resolve(wd)` and `fabricengine.RecordsWorktree`, as it did for the calibration.
Outside a resolvable worktree that resolution fails, so a run with the flag empty fails there, named slugs included, where named slugs ran before; passing the flag keeps working anywhere.
`checkHubMatchesWorktree` makes both directories absolute and cleaned with `filepath.Abs` and `filepath.Clean`, and returns an error naming the repository flag as `"-" + repositoryFlag` when `hub` differs from the parent of `wd`; a trailing separator and a relative spelling of the same directory are the same directory.
It is the guard that stops a slug being read against the wrong hub's weft: a hub flag naming another hub with the default weft would pair that hub's slugs with this hub's archive tags and plan commits.
The guard fires only when the repository flag is empty, so an explicit value skips it, and what it lets through is only a hub spelled differently from the parent of the current worktree, which it compares as paths and does not resolve through symlinks.
`run` then calls `archiveTags` once on the weft, hoisted before the slug handling, and passes the map to `recentRuns` when no slugs are named; after each `CountRun` it calls `ReadShedRecord(*hub, slug, tags[slug], gitrepo.New(*weft))` and `r.SetTime(rec)`, and a record error aborts the report with that error.
`recentRuns` takes the tag map in place of the slug set and tests a slug's presence in it, so `TestRecentRuns` changes its `finished` literal to the new map type; its one caller is `run`.
`finishedRuns` and `archivedSlugs` listed slugs only and are deleted with `TestArchivedSlugs`, whose slug-set checks `TestParseArchiveTags` from Card 3 now covers with the date field; the one caller of `finishedRuns` is `run`, and of `archivedSlugs` `finishedRuns` and that test.
`count.go` imports `gitexec` for `finishedRuns` alone, so that import is dropped with it.
A named slug whose worktree `<hub>/<slug>` still exists is in flight, so `ReadShedRecord` reads no tag for it and its section says no shed status was read.
The report title becomes `# Token usage and time by role`, and the output path `.scratch/token-usage-by-role.md` stays in the launchers; the repository flag's help text says the default applies whenever the flag is empty and is refused for a foreign hub.
`checkHubMatchesWorktree` lives in `main.go` beside `run`.
`TestCheckHubMatchesWorktree`, in `main_test.go`, is one test; the working directory is `filepath.Abs(".")` and never `os.Getwd`.
Its table rows call the helper: the hub is the parent of the working directory, so no error; the same hub with a trailing separator, no error; the relative spelling `..` against the working directory `filepath.Abs(".")`, no error; a temporary directory that is not the parent returns an error containing `"-" + repositoryFlag`; the working directory itself as hub returns that error.
One further subtest calls `run` with a hub flag naming a temporary directory, no repository flag and a slug, and asserts the error contains `"-" + repositoryFlag` and nothing is written; `run` reaches the guard after `lyxcwd.Getwd` and before `lyxcwd.Resolve`, so the row needs no worktree and pins that `run` calls the guard when the flag is empty.
The explicit-flag skip is pinned by `TestRunReportsShedTime`, which passes a temporary hub that is not the parent of the working directory together with the flag and still gets a report.
`shedstatus_integration_test.go` starts with the `//go:build integration` line like `calibrate_integration_test.go` and holds `TestRunReportsShedTime`, which runs `run` over a git-backed weft fixture built with `gitkit`, the only test that reaches the real `for-each-ref` committer-date format and the real `FileAtRevision` that the `fakeHistory`-backed unit rows cannot: it commits twice with `GIT_COMMITTER_DATE` set through `t.Setenv` (2026-01-01T12:00:00Z and 2026-01-02T12:00:00Z), the first commit holding an older run's `_lyx/shed/alpha/status.json`, the second holding alpha's current `seed.json` (`started_at` 2026-01-02T10:00:00Z) and `status.json` whose history is Plan-Write at 10:20:00Z, Plan-Review at 10:30:00Z folded with `repeats` 1 and `last_at` 10:40:00Z, and Publish at 11:00:00Z.
The second commit also holds beta's `_lyx/shed/beta/seed.json` and `status.json` (a one-entry history), so beta's tag has a readable status that only the in-flight rule can suppress.
Tags `archive/alpha/aaaaaaaaaaaa` and `archive/alpha/bbbbbbbbbbbb` point at the two commits, and `archive/beta/cccccccccccc` at the second.
A third commit, dated 2026-01-03T12:00:00Z, holds `_lyx/shed/gamma/status.json` that is not JSON and is tagged `archive/gamma/dddddddddddd`; a second `run` call naming `gamma` alone must return an error containing `gamma`, `archive/gamma/dddddddddddd` and `status.json`, which pins that a record error aborts the report.
The hub directory holds a `beta` directory, the in-flight worktree; alpha's one session is titled `ly:alpha:burler` and works 10:05 to 10:15 and 10:50 to 10:55, each turn ending on an `end_turn` assistant line written with Card 2's `stampedLine`, and beta and gamma have any timestamped session.
`run` is called with `"-" + repositoryFlag` set to the fixture, the hub flag the temporary hub, the projects flag the temporary projects directory and the slugs `alpha beta`, and the report must contain the title; alpha's span of `1h0m0s`, the burler row's `15m0s` and `25.0%`, the idle row's `45m0s` and `75.0%`; the Producers rows `| Plan-Review | 1 | 20m0s | 20m0s | 33.3% |`, `| Plan-Write | 1 | 20m0s | 10m0s | 33.3% |` and `| Publish | 1 | 20m0s | 15m0s | 33.3% |` in that order, which proves the newest tag was read and the older run's status was not; and under beta's heading the no-shed-status line and no Producers table, which proves the existing worktree suppressed beta's tag.
The tokencount row in `test-budget.yaml` rises by one net: two tests added, `TestArchivedSlugs` removed.

**ImpactSummary:** `run` has no caller outside `main` and `main_test.go`; the repository-flag default and the foreign-hub refusal change what a named-slug invocation needs, as the record states, and existing tests pass the flag explicitly or fail on flag parsing first.
