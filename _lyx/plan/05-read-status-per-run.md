# Card 5 — read-status-per-run

**Create:**
- `plan:tools/tokencount#TestRunReadsArchivedStatus` -> `func TestRunReadsArchivedStatus(t *testing.T)`

**Rename:**
- `tools/tokencount#TestRunRefusesTheRetiredHistoryFlag` -> `plan:tools/tokencount#TestRunRefusals`

**Edit:**
- `tools/tokencount#run`
- `tools/tokencount#repositoryFlag`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:tools/tokencount#readArchivedStatus`
- `plan:internal/shedrun#TreeSeedRel`
- `plan:internal/shedrun#TreeStatusRel`

**Intent:**
Wire the archived status into every counted run (Decision 8) and retitle the report (Decision 10).

`run` changes:
- The weft default, `fabricengine.RecordsWorktree` of the resolved current worktree, now applies whenever the repository flag (`repositoryFlag`) is empty, not only with no slugs or with `-calibrate`; outside a resolvable worktree `lyxcwd.Resolve` fails and `run` returns that error, named slugs included.
- Before deriving the default: when `-hub` was given, the repository flag is empty, and `filepath.Abs` + `filepath.Clean` of `-hub` differs from `filepath.Dir(wd)`, cleaned, `run` refuses with an error naming `-hub`'s directory, saying it is not the current worktree's hub, and naming the repository flag as the way forward, spelled `"-" + repositoryFlag`.
  This bounds the new default: it reads only the current hub's weft, and a foreign hub needs its weft named.
  A trailing slash or a relative spelling of the current hub cleans to the same path and is not refused.
- `finishedRuns` runs once on the weft, before the slug list is resolved, and serves both `recentRuns` and the status reads.
- For each slug, after `CountRun`: when `<hub>/<slug>` exists (`os.Stat` succeeds) the run is in flight, since teardown removes the worktree, so no tag is read and `Status` stays nil, the no-status fallback; this bounds the slug-reuse case where an older run's tag would otherwise stand in for a live run.
  Otherwise `run.Status` is `readArchivedStatus(gitrepo.New(weft), slug, tags[slug])`, and its error aborts the report before any output.
  The weft is only read, through `gitrepo`'s read side and the existing `for-each-ref` read (`PATTERN-fabric-git`, `PATTERN-gitexec-checked-call`).
- The repository flag's usage string, and the `repositoryFlag` comment, add the runs' archived shed status and seed to what the repository holds.
- The title line becomes `# Token usage and time by role`; the `-out` path the launchers pass stays `.scratch/token-usage-by-role.md`.

`TestRunRefusesTheRetiredHistoryFlag` becomes `TestRunRefusals`, a tier-1 table of `run` invocations that are refused with no report written:
- the retired `-history` flag, as today;
- `-hub` naming a fresh `t.TempDir()`, which is not `filepath.Dir` of the package directory the test runs in, with no repository flag: the error names the repository flag; the refusal fires before `lyxcwd.Resolve` and any git read.
`main_test.go`'s header comment says the file checks `run`'s refusals.

`TestRunReadsArchivedStatus` lands in a new integration-tier file `tools/tokencount/status_integration_test.go` (`//go:build integration`), not parallel since it calls `t.Chdir` and `t.Setenv`.
It builds a weft fixture with `gitkit` whose commits write each run's records at `shedrun.TreeSeedRel(slug)` and `shedrun.TreeStatusRel(slug)` and are tagged `archive/<slug>/<first 12 hex>`, each commit given its own `GIT_COMMITTER_DATE` through `t.Setenv`, which reaches gitkit's git calls:
- `alpha`: an older commit whose seed has no `started_at`, tagged lightweight as `archiveWeftTip` tags, then a newer commit whose seed has `started_at` and whose status has two history entries, tagged annotated (`tag -a`) so the peeled fields of `finishedRuns`' format are exercised against real git.
  The dates are ordered older commit < `started_at` < first entry end < second entry end, so neither bound clamps the start, and the newer commit's date is later than the older one's.
- `live`: a tagged status, and a directory `<hub>/live`.
- `broken`: a tagged `status.json` that is not JSON.
Every slug gets one session transcript under `projectDir(projects, <hub>/<slug>)`, so `CountRun` succeeds and `run` reaches the status read.
Then, with the working directory changed to the weft fixture:
- `run` with the repository flag naming the fixture, `-hub`, `-projects` and `alpha live`: the report opens with `# Token usage and time by role`, `alpha`'s section carries a `### Producers` table whose elapsed figures sum to the span from the newer seed's `started_at` to the second entry's end, and `live`'s section carries the no-status line though its tag exists;
- `run` with `broken`: an error naming `broken` and an empty output, the abort; the error's wording is pinned by card 3's `readArchivedStatus` rows.
Then, with the working directory changed to a fresh `t.TempDir()`, which is in no git repository:
- `run` with `-hub` spelled as that directory's parent with a trailing slash, and again as `..`, and no repository flag: the error is not the foreign-hub refusal but the `lyxcwd.Resolve` failure, since no default weft can be derived outside a worktree.
`TestCalibrateGitBackedMatchesInMemory` keeps passing unchanged: it names the repository flag, its fixture has no archive tag and `<hub>/alpha` does not exist, so `alpha` takes the no-status fallback.
The package's row in `cmd/lyx/testdata/test-budget.yaml` rises by one for `TestRunReadsArchivedStatus`; the rename keeps the count.

**ImpactSummary:** `run` is the tool's only entry point and `repositoryFlag` a comment-only change; the behavior change is that a run with the repository flag empty now always derives the weft, or is refused for a foreign `-hub`, and the renamed test has no other reference.
