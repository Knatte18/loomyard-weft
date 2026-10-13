# Card 5 — wire-run-time

**Delete:**
- `tools/tokencount#TestRunRefusesTheRetiredHistoryFlag`

**Create:**
- `plan:tools/tokencount#TestRunArguments` -> `func TestRunArguments(t *testing.T)`

**Edit:**
- `tools/tokencount#run`
- `tools/tokencount#TestCalibrateGitBackedMatchesInMemory`

**Uses:**
- `tools/tokencount#finishedRuns`
- `plan:tools/tokencount#readShedRecord`
- `plan:tools/tokencount#RunTally.MeasureTime`
- `plan:internal/shedrun#RunRecordRels`
- `internal/gitkit#Git`

**Intent:**
Wire the archived shed record into every run the tool counts, make the weft default apply whenever the weft flag (`repositoryFlag`) is empty, bound that default with a refusal, and retitle the report (Decisions 8 and 10).

`run`:
- After `wd` and the `-hub` default, when the weft flag is empty, `run` compares `-hub` with `filepath.Dir(wd)`, the `-hub` default and the current worktree's parent when run from its root as the launchers do, both made absolute and cleaned with `filepath.Abs`.
  When they differ it refuses before resolving or reading anything, with the error "-hub <hub> is not the parent of the current worktree <dir>, so the default weft, that worktree's weft sibling, would read another hub's archive tags; name the weft repository of <hub> with the weft flag", the flag spelled through `repositoryFlag` with its leading dash.
  The input that reaches it: `-hub` naming any other directory with the weft flag empty, whether slugs are named or not; `TestRunArguments` reaches it.
  The same hub spelled with a trailing slash or as a relative path is not refused, since `filepath.Abs` cleans both.
- Otherwise, when the weft flag is empty, it resolves `lyxcwd.Resolve(wd)` and takes `fabricengine.RecordsWorktree` of it, now whatever the slugs and `-calibrate`.
  What this lets through: a run with named slugs and no `-calibrate`, which read no repository before, now reads the current hub's weft; the refusal above bounds it to the hub the current worktree belongs to.
  Outside a resolvable worktree the default cannot be derived, so such a run now fails with `lyxcwd`'s error where it ran before; the weft flag is the way past it.
- It calls `finishedRuns` once, before the slug choice, and keeps the map: `recentRuns` gets it as before, and each counted slug's tags come from it.
- One `gitrepo.New(*weft)` serves the shed records and, under `-calibrate`, `Calibrate` and `BuildProfileReport`, replacing the `fabric` handle it builds today.
- For each slug, after `CountRun`, it calls `readShedRecord(records, *hub, slug, finished[slug])`, returns its error unchanged (it names the slug, the tag and the file), and calls `MeasureTime` with the record before appending the run, so `CountOrch` and the report see measured runs.
- The report's first line becomes `# Token usage and time by role`; the output path the launchers pass stays `.scratch/token-usage-by-role.md`.

`TestRunArguments` in `main_test.go` replaces `TestRunRefusesTheRetiredHistoryFlag`, so the refusals of `run`'s arguments share one table-driven test and the package's test count stays at card 2's budget.
It changes the working directory with `t.Chdir` to a fresh `<tmp>/worktree`, so it never runs in parallel, and says so in a comment.
Each row passes `-projects` a temporary directory and one slug, so no row reads the operator's transcripts.
Rows:
- `-history somewhere -calibrate fit slug`: the undefined-flag error naming `-history`, as the deleted test asserted.
- `-hub <another temp dir>`, the weft flag empty: exactly the refusal text above, naming both directories and the weft flag.
- `-hub <tmp>/` with a trailing slash and `-hub ..`, both the current worktree's parent: an error (the temporary directory is no worktree, so `lyxcwd.Resolve` fails) that is not the refusal.
Every row asserts that nothing was written to the output.
Update `main_test.go`'s file comment, which names only the retired history flag, to say it checks `run`'s argument refusals.

`TestCalibrateGitBackedMatchesInMemory` gains the run's archived shed records: before its `run` call it commits a `seed.json` with `started_at` 10 minutes before the fork's start and a `status.json` whose history holds a `Plan-Write` entry finishing 5 minutes before the fork's start and a `Webster` entry finishing 5 minutes after it, at the paths `shedrun.RunRecordRels("alpha")` gives, then tags that commit `archive/alpha/<first 12 hex of its SHA>` with `git tag`.
The stamps are written as RFC 3339 from the fork's start truncated to the second, so every duration is exact.
The report it already checks must then also hold `# Token usage and time by role`, the run's span sentence naming that tag, and the Producers rows `| Plan-Write | 1 | 5m0s | 5m0s | 33.3% |` and `| Webster | 1 | 10m0s | 10m0s | 66.7% |`, since the fixture's transcripts hold no working interval.
This pins the `for-each-ref` format and the blob read against real git.
Update the test's comment above the `run` call to say `run` also reads the run's archived shed status from the fixture.

**ImpactSummary:** `run` is the tool's only entry; it gains a refusal for a foreign `-hub` with the weft flag empty and now needs a resolvable worktree or the weft flag for named slugs too, and the deleted test's one row moves into `TestRunArguments`.
