# Card 5 — run-wiring

**Create:**
- `plan:tools/tokencount#foreignHub` -> `func foreignHub(hub, wd string) bool`
- `plan:tools/tokencount#TestRunRefusals` -> `func TestRunRefusals(t *testing.T)`
- `tools/tokencount/status_integration_test.go`

**Delete:**
- `tools/tokencount#finishedRuns`
- `tools/tokencount#archivedSlugs`
- `tools/tokencount#TestArchivedSlugs`
- `tools/tokencount#TestRunRefusesTheRetiredHistoryFlag`

**Edit:**
- `tools/tokencount#run`

**Uses:**
- `plan:tools/tokencount#readArchivedRun`
- `plan:tools/tokencount#archiveTags`
- `plan:tools/tokencount#TestParseArchiveTags`
- `plan:internal/shedrun#SeedTreeRel`
- `plan:internal/shedrun#StatusTreeRel`

**Intent:**
Read every counted run's archived status in `run` and retitle the report (Decisions 3, 8 and 10).
"The repository flag" below is the flag `repositoryFlag` names.

- The repository flag's default, the current worktree's records sibling, now applies whenever the flag is empty, not only with no slugs or with `-calibrate`.
  What that lets through: with `-hub` naming another hub, the current hub's records would be read for that hub's slugs.
  The bound, checked before `lyxcwd.Resolve` so it needs no worktree: when the repository flag is empty, `-hub` is given and `foreignHub(*hub, wd)` holds, `run` returns an error naming `-hub`'s value, the current worktree's parent, and the repository flag as the way forward, built from `repositoryFlag` so the name is spelled once.
  `wd` is the value `run` already takes from `lyxcwd.Getwd`.
  `foreignHub` resolves a relative `hub` against `wd`, cleans both sides and compares the result with `filepath.Dir(wd)`, so `<parent>/` and `..` from the worktree are the current hub.
  Outside a resolvable worktree, an empty repository flag now fails at `lyxcwd.Resolve` with named slugs too, as the record states.
- With the repository always set, `run` lists the tags once with `archiveTags` and uses its keys as the finished set for the no-slug path.
  `finishedRuns` and `archivedSlugs`, now unused, are deleted with `TestArchivedSlugs`, whose input `TestParseArchiveTags` already carries, and `count.go` drops its `gitexec` import, which only `finishedRuns` used.
- What the status read lets through: a named slug in flight would take an older run's archive tag of the same slug.
  The bound: after `CountRun`, a slug whose worktree `<hub>/<slug>` still exists (`os.Stat` succeeds) is in flight and reads no tag; any other slug calls `readArchivedRun` over `gitrepo.New` of the repository with its tags, then `setSpan` with the result, so a slug without a tag, or in flight, keeps `CountRun`'s transcript-window span and its section says no shed status was read.
  An error from `readArchivedRun` aborts the report.
- The report title becomes `# Token usage and time by role`; the output path is unchanged.

Tests:
- `TestRunRefusals`, in `main_test.go`, tier 1, replaces `TestRunRefusesTheRetiredHistoryFlag` with a table of `run` refusals, each asserting the error text and that no report is written:
  - the retired `-history` flag, the existing case unchanged;
  - `-hub <t.TempDir()>` with no repository flag and one slug: the error contains the repository flag's name, the temp directory and `filepath.Dir` of the test's `lyxcwd.Getwd`;
  - one slug, no `-hub` and no repository flag, run from a `t.TempDir()` cwd set with `t.Chdir`: `run` fails at `lyxcwd.Resolve`, and the error is not the foreign-hub refusal.
  
  Rows over `foreignHub` pin that a temp directory is foreign while `filepath.Dir(wd) + "/"` and `".."` with the test's `wd` are not.
  The test is not parallel, since one row changes the cwd; a comment says so.
- `status_integration_test.go`, `//go:build integration`, adds `TestRunReadsArchivedStatus`.
  It is not parallel, since it sets `GIT_COMMITTER_DATE` with `t.Setenv` to order its commits.
  A hermetic repository built with `gitkit` in a `t.TempDir()` holds, at paths from `shedrun.StatusTreeRel` and `SeedTreeRel`, an older commit tagged `archive/alpha/ffffffffffff` with one status, a newer commit tagged `archive/alpha/000000000000` whose status has a different history and whose seed has `started_at`, and a commit tagged `archive/beta/<sha>`.
  The older commit's tag sorts last by name, so only a sort by committed time picks the newer one.
  The test hub holds the directory `beta`, so beta is in flight, and no tag names `gamma`.
  Each slug has a session transcript under `-projects`.
  `run`, given that repository through the repository flag, `-hub`, `-projects` and the slugs `alpha beta gamma`, writes the new title, alpha's span line naming the newer tag, the Producers rows of the newer history and not the older, and the no-status line and no `| producer |` header in both the beta and gamma sections.
- The `tools/tokencount` budget row needs no change: two tests are deleted and two created.

`TestCalibrateGitBackedMatchesInMemory`'s `run` passes the repository flag, its repository holds no `archive/` tag and its hub no `alpha` directory, so it passes unchanged.

**ImpactSummary:** confined to `run` in `tools/tokencount/main.go` and the two tag helpers it alone called; the only behaviour change outside the report's content is the new `-hub` refusal and the defaulted repository flag for named slugs.
