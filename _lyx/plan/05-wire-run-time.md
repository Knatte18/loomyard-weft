# Card 5 — Wire the archived status into every run, bound -hub, document the time report

**Rename:**
- `tools/tokencount#TestRunRefusesTheRetiredHistoryFlag` -> `plan:tools/tokencount#TestRunRefusesFlags`

**Create:**
- `plan:tools/tokencount#errHubNeedsWeft` -> `var errHubNeedsWeft = errors.New("")`

**Edit:**
- `tools/tokencount#run`
- `tools/tokencount/tokencount.sh`
- `tools/tokencount/tokencount.cmd`
- `plan:tools/tokencount#TestShedTimeFromGitArchive`

**Delete:**
- `tools/tokencount#finishedRuns`
- `tools/tokencount#archivedSlugs`

**Intent:**
Decisions 8 (the weft default, the -hub bound, the in-flight bound) and 10, and the scope's documentation item.
Below, "the weft flag" is the flag `repositoryFlag` names, -weft on the command line.

`run` in `main.go`:
- The weft default, `fabricengine.RecordsWorktree` of `lyxcwd.Resolve(wd)`, applies whenever the weft flag is empty, not only with no slugs or with `-calibrate`; a `Resolve` failure is returned as now, so outside a resolvable worktree a run with the weft flag empty fails, named slugs included.
- Before that default, when `-hub` was given and the weft flag is empty, `run` compares `filepath.Abs(*hub)` (which cleans) with `filepath.Dir(wd)`; when they differ it returns `fmt.Errorf("%w: %s is not %s", errHubNeedsWeft, abs, filepath.Dir(wd))` and writes no report.
  `errHubNeedsWeft`, declared in `main.go`, says in its text that -hub names another hub's directory, so -weft must name that hub's weft repository.
  What this guard lets through: -hub spelled as the current worktree's parent in any form that cleans to it (a trailing slash, a relative path), and any -hub paired with an explicit weft flag, whose pairing the operator then owns; it refuses only the case where the default would read the wrong hub's records.
- The no-slug branch calls card 3's `archiveTags` where it called `finishedRuns`, and builds the `map[string]bool` that `recentRuns` takes from the map's keys.
  When slugs are named, `run` calls `archiveTags` once after the slug loop.
  `finishedRuns` and `archivedSlugs` lose their last callers (`TestArchivedSlugs` moved to `parseArchiveTags` on card 3) and are deleted.
- After the slug loop, for each counted run, `run` calls `readShedTime(gitrepo.New(weft), tags[slug], inFlight)` on `report.Runs[i]`, where `inFlight` is true only for a named slug whose worktree `filepath.Join(*hub, slug)` exists (`os.Stat` succeeds); auto-picked slugs are never in flight.
  An error from `readShedTime` is returned, so a seed or status that fails to decode aborts the report naming the slug, the tag and the file.
  The tag read comes after the slug loop, so a named slug with no sessions still fails on `CountRun` before any git read.
- The report title becomes `# Token usage and time by role`; the output path the launchers pass, `.scratch/token-usage-by-role.md`, stays.
- The weft flag's usage text says its default applies whenever the flag is empty.

The package doc at the top of `main.go` (one sentence per line, `PATTERN-comment-line-breaks`):
- The opening sentence says the report holds token use and time per run and per role.
- The weft flag's sentence says its default applies whenever the flag is empty, that a -hub other than the current worktree's parent then needs the flag, and that outside a resolvable worktree the flag must be given.
- A new paragraph after the share column's describes time: working time is turn time (Decision 1's turn-ending lines: an assistant line whose `stop_reason` is set and not `tool_use`, a user line starting with `[Request interrupted by user`, a user line starting with `<local-command-stdout>`); a role's time is the union of its transcripts' working intervals clipped to the run's span, `+sub` its own row; the span runs from the seed's `started_at`, else the earliest transcript line, bounded below by the slug's next-older archive tag's commit date and above by the first history entry's end, to the latest history entry's end, a folded entry ending at its `last_at`; the run's tag is the slug's newest-committed archive tag, read through `gitrepo`, and a named slug whose worktree still exists is in flight and reads none; a run with no tag or no status uses its transcripts' window and says no shed status was read; the `idle` row, the `time` and `time share` columns and the blank orch time cells; and the Producers tables, each history entry owning the time from the latest earlier finish to its own, with elapsed summing to the span.
- The sentence defining a finished run keeps its meaning.

The launchers' description comments, `tokencount.sh` and `tokencount.cmd`, say the report gives token use and time by role: the time columns, the idle row and the Producers tables.

Tests:
- `TestRunRefusesTheRetiredHistoryFlag` becomes the table-driven `TestRunRefusesFlags`, each row calling `run` with `-projects` set to a fresh temporary directory and asserting the error and that nothing was written to stdout:
  - the retired `-history` flag: the undefined-flag error, as now;
  - -hub a temporary directory and no weft flag: `errors.Is(err, errHubNeedsWeft)`;
  - -hub a temporary directory with the weft flag a temporary directory and slug `nope`: not `errHubNeedsWeft`, but the `no sessions for nope` error of `CountRun`;
  - `-hub ..` (the test's working directory is the package directory, so `..` is its parent) and slug `nope`: an error that is not `errHubNeedsWeft`;
  - -hub the absolute parent with a trailing separator and slug `nope`: the same.
  The last two rows pass the guard and then fail on whichever comes first, `lyxcwd.Resolve` in a checkout that does not resolve or `CountRun`'s `no sessions for nope`, both before any git read, so they assert only that the guard let them through.
  The file comment of `main_test.go` is reworded to say it checks the flag refusals: the retired history flag, and -hub naming another hub with no weft flag.
- `TestShedTimeFromGitArchive` gains a block driving `run` end to end, with its repository as the weft flag, a temporary hub as -hub and a temporary `-projects` holding sessions for two named slugs that both have an archive tag with a status: `run-x`, whose worktree directory is absent, and `run-y`, whose worktree directory under the hub exists.
  The report written to stdout holds the title `# Token usage and time by role` (no other test reaches the title), a `### Producers` table in `run-x`'s section, the sign that `run` handed it its tag, and the `No shed status was read` line in `run-y`'s section, the sign that the in-flight check skipped its tag.
  A second `run` call names a third slug `run-z`, whose tag holds a `seed.json` with an unknown key: it returns an error naming `run-z` and `seed.json`, and writes nothing to stdout.

**ImpactSummary:** `run` is the tool's one entry point; with the weft flag empty it now resolves the weft for named slugs too and refuses another hub without the flag, and every report gains the time sections; `finishedRuns` and `archivedSlugs` have no caller left; no package outside `tools/tokencount` is touched.

**Commit:** 5: tokencount: read every run's archived status, bound -hub, document time
