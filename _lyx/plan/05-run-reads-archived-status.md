# Card 5 — run measures every run's time from its archived status

**Create:**
- `tools/tokencount/runtime_integration_test.go`
- `plan:tools/tokencount#checkHubWeft` -> `func checkHubWeft(hub, weft, wd string) error`
- `plan:tools/tokencount#TestHubNeedsWeft` -> `func TestHubNeedsWeft(t *testing.T)`
- `plan:tools/tokencount#TestRunReadsArchivedStatus` -> `func TestRunReadsArchivedStatus(t *testing.T)`

**Edit:**
- `tools/tokencount/main.go`
- `tools/tokencount/main_test.go`
- `tools/tokencount/tokencount.sh`
- `tools/tokencount/tokencount.cmd`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:tools/tokencount#readShedRecord`
- `plan:tools/tokencount#RunTally.MeasureTime`
- `plan:tools/tokencount#ShedRecord`

**Intent:**
Decisions 8 and 10, the wiring: `run` reads every counted run's archived status and measures its time.
The new title and the docs describe what the report now holds.

`checkHubWeft(hub, weft, wd)`, in `tools/tokencount/main.go`, returns nil when `weft` is non-empty.
Otherwise it makes `hub` absolute by joining it onto `wd` when relative, never through `filepath.Abs`, which calls `os.Getwd` (`PATTERN-cwd-resolution`).
It cleans that path and compares it with `filepath.Clean(filepath.Dir(wd))`.
A mismatch returns an error reading `-hub <hub> is not the current worktree's hub <parent>; name that hub's records repository with` followed by the weft flag's name, -weft, spelled through the existing `repositoryFlag` constant.

The card lists `tools/tokencount/main.go` as a whole file, since it changes most of it: the package doc, `run` and the new `checkHubWeft`.

`run` changes in order:
- After `-hub` defaults to `filepath.Dir(wd)`, it calls `checkHubWeft(*hub, *weft, wd)` and returns its error before any read.
- -weft, when empty, now always defaults to `fabricengine.RecordsWorktree` of `lyxcwd.Resolve(wd)`, not only with no slugs or with `-calibrate`.
  Outside a resolvable worktree `lyxcwd.Resolve` fails and `run` returns that error, named slugs included, where named slugs ran before.
- `finishedRuns(*weft)` now runs on every invocation, since each run needs its tags.
  The no-slug path keeps using its keys for `recentRuns`.
- For each slug after `CountRun`, its record depends on whether `filepath.Join(*hub, slug)` exists.
  When it does (`os.Stat` succeeds), the run is in flight, since teardown removes the worktree, and its record is the zero `ShedRecord`, so no archive tag is read.
  Otherwise the record is `readShedRecord(gitrepo.New(*weft), slug, tags[slug])`, whose error `run` returns.
  `run` then calls `MeasureTime(record)` and appends the run.
  The in-flight check applies to named slugs only, as Decision 8 bounds it (slugs were named when `fs.Args()` is non-empty); a slug the default listing picks always reads its newest tag.
- The report title becomes `# Token usage and time by role`; the launchers' `-out` path `.scratch/token-usage-by-role.md` stays.

What the change lets through, and its bounds:
- The weft default now serves named slugs, which could read a slug against the wrong hub's records.
  `checkHubWeft` bounds that: a `-hub` naming another directory than the current worktree's parent needs -weft.
- Reading status for a named slug could hand an in-flight run an older run's archive tag.
  The worktree check bounds that.

`tools/tokencount/main_test.go`'s file comment, which names only the retired history flag, now also names the foreign-hub refusal.

The package doc in `tools/tokencount/main.go` keeps one sentence per line (`PATTERN-comment-line-breaks`) and is updated in place:
- Its opening sentence says the report holds token use and time per run and per role.
- The two -weft sentences say the default applies whenever -weft is empty, and that a `-hub` other than the current worktree's parent needs -weft.
  They also say a run with -weft empty fails outside a resolvable worktree.
- The sentence on the orch row says each table ends with the orch row, then the `idle` row.
- A new paragraph covers the `time` and `time share` columns, the `idle` row and the per-run and "All runs" Producers tables, and how each is measured (Decisions 1 to 5 and 7).
  It also covers how the span's records are read:
  - the run's newest `archive/<slug>/<sha>` tag by committer date;
  - the seed and status read through `gitrepo` at that commit;
  - the start bound at the next-older tag's commit time and at the first history entry's end;
  - an in-flight worktree reading no tag;
  - the transcript-window fallback, with its note;
  - a decode failure aborting with the slug, the tag and the file named.

The description comments in `tokencount.sh` and `tokencount.cmd` say the report is token usage and time by role, with each role's working time, the run's idle time and the Producers split.

Tests:
- `TestHubNeedsWeft`, new in `tools/tokencount/main_test.go`, is a table over `checkHubWeft` with `wd` `/x/hub/prime`:
  - `/x/other` with -weft empty is refused, and the error contains -weft;
  - `/x/hub/` with a trailing slash passes;
  - `..`, relative to `wd`, passes;
  - `../../elsewhere` is refused;
  - `/x/other` with -weft set passes.
  It then runs `run([]string{"-hub", t.TempDir(), "slug"}, &out)` from the package directory.
  It asserts an error containing -weft and an empty `out`, the refusal reached through `run` before any read.
- `TestRunReadsArchivedStatus`, in the new `tools/tokencount/runtime_integration_test.go` under `//go:build integration`, does not run in parallel: it calls `t.Chdir` and `t.Setenv`.
  It builds a weft repository with `gitkit`, each commit's committer time set through `t.Setenv("GIT_COMMITTER_DATE", ...)`, and the record paths from `shedrun.TreeRecordRels`:
  - An `alpha` status naming a producer `Old` at a commit dated 09:00, tagged `archive/alpha/ffffffffffff`.
  - A newer `alpha` status and a seed without `started_at` at a commit dated 12:00, tagged `archive/alpha/000000000000`.
    The tag names sort opposite to their commit times, so picking by name instead of by committer date fails the test.
  - A `beta` status at a third commit dated 11:00, tagged `archive/beta/<sha12>`, while the directory `<hub>/beta` exists.
  Transcripts come from `writeLines` and `timedLine` under a temporary `-projects`.
  `alpha` gets an older session line at 08:00 and run sessions working from 10:00.
  `beta` gets one session working 10:00 to 10:20, since `CountRun` refuses a slug with no session.
  The prime, `<hub>/prime`, holds an `ly:orch` session whose sub-agent's first prompt names `alpha` and works in a stretch where no `alpha` session works.
  It runs `run` from `<hub>/prime` with -weft, `-hub` and `-projects` and the slugs `alpha beta`, and asserts:
  - the title `# Token usage and time by role`;
  - `alpha`'s `Span` line starting at 09:00, the older tag's commit time;
  - `alpha`'s Producers rows from the newer status, with no `Old` row;
  - `alpha`'s `idle` row equal to the span minus the `alpha` sessions' union, so the orch sub-agent's working stretch counts as idle;
  - `beta`'s section carrying `No shed status was read; the span is the transcript window.` and no Producers table.
  Last, it runs `run` with -weft empty, `-hub` omitted, `-projects` as before and the named slug `alpha` from the same non-git directory.
  That is the case that ran before this card.
  It asserts no report and an error whose text equals that of `lyxcwd.Resolve` called on the same directory, so a failure from any later step fails the row.
- The two new top-level tests raise the `tools/tokencount` row of `cmd/lyx/testdata/test-budget.yaml` from 14 to 16.

**ImpactSummary:** `run` is the tool's only entry point; the report gains its time content for real runs and the new title, a named-slug invocation now needs a resolvable worktree or -weft, and a foreign `-hub` without -weft is refused; no other package calls into `tools/tokencount`.

**Commit:** 5: tokencount: measure each run from its archived shed status
