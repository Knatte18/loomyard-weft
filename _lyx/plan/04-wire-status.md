# Card 4 — Read every run's archived status from the records repository

**Create:**
- `plan:tools/tokencount#checkHubRepository` -> `func checkHubRepository(hub, wd string) error`
- `plan:tools/tokencount#TestCheckHubRepository` -> `func TestCheckHubRepository(t *testing.T)`

**Edit:**
- `tools/tokencount#run`
- `tools/tokencount#TestCalibrateGitBackedMatchesInMemory`
- `tools/tokencount/tokencount.sh`
- `tools/tokencount/tokencount.cmd`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:tools/tokencount#attachStatus`
- `plan:tools/tokencount#ArchiveTag`
- `plan:tools/tokencount#producerRule`
- `plan:internal/shedrun#StatusTreeRel`
- `plan:internal/shedrun#SeedTreeRel`
- `tools/tokencount#repositoryFlag`
- `tools/tokencount#finishedRuns`
- `tools/tokencount#recentRuns`
- `internal/lyxcwd#Getwd`
- `internal/gitrepo#New`

**Intent:**
Decision 8's wiring: every run's status is read from the records repository the flag named by `repositoryFlag` gives, whether or not slugs are named, with the two bounds the decision sets.
Below, "the repository flag" is that flag.

`run`:
1. When the repository flag is empty, it first calls `checkHubRepository(*hub, wd)`, then derives the default from `lyxcwd.Resolve(wd)` and `fabricengine.RecordsWorktree` as it does now; the condition `len(slugs) == 0 || *calibrate != ""` is dropped, so the default applies whenever the flag is empty.
   Outside a resolvable worktree `lyxcwd.Resolve` fails, so a run with the flag empty and named slugs now fails there, where it ran before; that is the decision's stated cost.
2. It always calls `finishedRuns` on that repository, and with no slugs passes the result to `recentRuns` as now.
3. It builds one `gitrepo.New` over that repository, used for the status reads and, with `-calibrate`, for `Calibrate` and `BuildProfileReport` as before.
4. After `CountRun` for each slug it calls `attachStatus(&r, *hub, tags[slug], records)` and returns its error, so a decode failure aborts the report.
5. The repository flag's help string adds each run's archived shed status to what the repository holds.

`checkHubRepository` is the bound on reading the wrong hub's records: the default is the current hub's records repository, so a `-hub` naming another hub with the repository flag empty is refused.
It compares `filepath.Abs(hub)` with `filepath.Clean(filepath.Dir(wd))` and, when they differ, returns an error saying `-hub` names a hub other than the current worktree's and that the records repository holding that hub's archive tags must be named with the flag, spelled `"-" + repositoryFlag`; an `Abs` failure is returned wrapped.
It skips nothing else: with the repository flag set it is never called, and the operator then owns the hub-repository pairing as before.
The input that reaches the refusal is `-hub` set to any directory but the current worktree's parent with the repository flag empty; `TestCheckHubRepository` reaches it.

The package doc in `main.go`:
- the sentences on the repository flag say its default applies whenever the flag is empty, that a `-hub` other than the current worktree's parent then needs the flag, and that every run's status is read from it, not only with no slugs or `-calibrate`;
- a new paragraph after the time paragraph card 2 added says how the span is measured (Decision 3: seed `started_at` or the earliest transcript line, bounded by the next-older archive tag's commit time and the first history entry's end, to the latest history entry's end, a folded entry ending at its `last_at`), which tag is read (the newest by committer time), that a run whose worktree `<hub>/<slug>` still exists is in flight and reads no tag, that a run without a status uses its transcript window and its section says so, and what the Producers table holds (Decision 5) and why its elapsed includes waits;
- the sentence saying a run still in flight would count as one whose later steps cost nothing stays.

`tokencount.sh` and `tokencount.cmd`'s description comments say the report holds token use and time by role: the time and time-share columns, the idle row and the per-run Producers table, read from each run's archived shed status in the records repository; the output path is unchanged.

Tests:
- `TestCheckHubRepository` goes in `main_test.go`, whose file comment then says it checks tokencount's flags: the retired history flag is refused, and so is a foreign `-hub` without the repository flag.
  It calls `checkHubRepository` directly, since `run` past the check resolves the real worktree and reads its records repository.
  It is table-driven with `wd` from `lyxcwd.Getwd()`: a `-hub` of another temp directory is refused with an error naming `"-" + repositoryFlag`; `filepath.Dir(wd)` with a trailing separator, and `..` (relative to the test's working directory, which is `wd`), are not refused.
- `TestCalibrateGitBackedMatchesInMemory`'s end-to-end part gains archived status for `alpha`: `started` is truncated to the second; before `run`, the fixture commits a status and seed for `alpha` at `shedrun.StatusTreeRel("alpha")` and `SeedTreeRel("alpha")`, with seed `started_at` ten minutes before the fork start and one Plan entry five minutes after it, and tags the commit `archive/alpha/<first 12 hex>`.
  The report then holds `# Token usage and time by role`, `| Plan | 1 | 15m0s |` and `producerRule`, which pins the real `for-each-ref` format through `run`; the hub directory holds no `alpha` worktree.
  The newest-tag choice is pinned by `TestArchivedSlugs` and `TestAttachStatus` in card 3.
- `cmd/lyx/testdata/test-budget.yaml`'s `tools/tokencount` row goes from 14 to 15 for `TestCheckHubRepository`.

**ImpactSummary:** Changes `tokencount`'s CLI behavior only: an empty repository flag now always reads the current hub's records and is refused with a foreign `-hub`; the integration test is the one existing test that drives `run` past flag parsing, and `TestRunRefusesTheRetiredHistoryFlag` fails at flag parsing before either change.

**Commit:** 4: tokencount reads every run's archived status from the records repository
