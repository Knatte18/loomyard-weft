# Card 3 — The run's span from its archived shed status

**Create:**
- `plan:tools/tokencount#archiveTag` -> `type archiveTag struct`
- `plan:tools/tokencount#entryEnd` -> `func entryEnd(e shedengine.HistoryEntry) (time.Time, error)`
- `plan:tools/tokencount#readArchive` -> `func readArchive(run *RunTally, tags []archiveTag, records PlanHistory) error`
- `plan:tools/tokencount#TestReadArchive` -> `func TestReadArchive(t *testing.T)`

**Edit:**
- `tools/tokencount#finishedRuns` -> `func finishedRuns(weft string) (map[string][]archiveTag, error)`
- `tools/tokencount#archivedSlugs` -> `func archivedSlugs(refs string) map[string][]archiveTag`
- `tools/tokencount#recentRuns` -> `func recentRuns(projects, hub, prime string, finished map[string][]archiveTag, last int) ([]string, error)`
- `tools/tokencount#run`
- `tools/tokencount#RunTally`
- `tools/tokencount#Report.WriteMarkdown`
- `tools/tokencount#TestArchivedSlugs`
- `tools/tokencount#TestRecentRuns`
- `tools/tokencount#TestCalibrateGitBackedMatchesInMemory`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `tools/tokencount#PlanHistory`
- `plan:internal/shedrun#ArchivedSeedRel`
- `plan:internal/shedrun#ArchivedStatusRel`
- `plan:internal/shedrun#DecodeSeed`
- `internal/shedrun#ValidateRunID`
- `internal/gitrepo#Repo.FileAtRevision`
- `internal/gitrepo#ErrPathNotAtRevision`

**Intent:**
Decisions 3, 8 and 9: a finished run's span runs from its start to the latest end among its archived shed history entries, read from the run's `archive/<slug>/<sha>` tag in the weft.

Tags.
`archiveTag` is one archive tag: `Ref` (the tag name `archive/<slug>/<sha>`), `SHA` (its commit's full object name) and `Committed` (the commit's committer time).
`finishedRuns` keeps its `gitexec.Run` `for-each-ref` call over `refs/tags/archive/` and asks for `--format=%(refname) %(objectname) %(committerdate:iso-strict)`; archive tags are lightweight (`fabricengine`'s `archiveWeftTip` runs `git tag <tag> <tip>`), so `objectname` is the commit.
`archivedSlugs` parses that output into each slug's tags, skipping a line that is not an archive tag with a slug and a sha, as now, or whose sha or date does not parse.
`recentRuns` counts a slug as finished when it has at least one tag; its behavior is otherwise unchanged.

Reading (Decisions 8 and 9).
`readArchive` reads through the existing `PlanHistory` interface, whose `FileAtRevision` is the one method it calls; the weft holds the plan commits and the archive tags alike, and `*gitrepo.Repo` and the tests' `fakeHistory` already satisfy it.
`readArchive` lives in a new `tools/tokencount/archive.go`.
It takes the newest of `tags` by `Committed` (ties broken by `Ref`) and, when there is one, the next older tag's `Committed` as the start's lower bound.
It reads `shedrun.ArchivedStatusRel(run.Slug)` and `shedrun.ArchivedSeedRel(run.Slug)` at the newest tag's `SHA`.
A read error other than `gitrepo.ErrPathNotAtRevision`, a seed `shedrun.DecodeSeed` refuses, a `status.json` that `json.Unmarshal` cannot decode into `shedengine.Status`, and a history entry whose end does not parse as RFC 3339 each abort with an error naming the slug, the tag and the file, as `<slug>: <tag>: <file>: <cause>`, the file being the relative path the shedrun pair returned.
A run id the path pair refuses has no path to name, and the tool spells no record file name itself (`PATTERN-shed-run-directory`), so it aborts as `<slug>: <tag>: <cause>`, the cause being `ValidateRunID`'s refusal, which names the run id.
A `status.json` absent from the tag, or one with no history entry, leaves the run without status: it keeps Decision 3's transcript-window span.
An absent `seed.json`, or a seed with an empty `StartedAt`, starts the span at `run.First`.

Span (Decision 3).
`entryEnd` is an entry's `LastAt` when `Repeats` > 0, else its `At`, parsed with `time.RFC3339`.
With status read, `SpanTo` is the latest entry end, and `SpanFrom` is the seed's `StartedAt` (an unparseable one aborts as above), else `run.First`; then raised to the older tag's `Committed` when it is earlier, then lowered to the first history entry's end when it is later.
`RunTally` gains `HasStatus bool`, `Tag string` (the read tag's `Ref`) and `History []shedengine.HistoryEntry`, all set only when status was read.

Run (Decision 8).
`run` compares `-hub` against the current worktree's parent, `filepath.Dir(wd)`, each made absolute against `wd` (joined onto it when relative, never through `filepath.Abs`, which calls `os.Getwd`; `PATTERN-cwd-resolution`) and cleaned; when they differ and the weft flag, the one `repositoryFlag` names, is empty it refuses before any git read with an error naming the hub, the parent and that flag as the way forward: name the hub's weft repository with it.
What the refusal bounds: the weft default now applies whenever the weft flag is empty, not only with no slugs or with `-calibrate`, so a slug read against another hub's records needs the weft flag named; the comparison lets a trailing slash or a relative spelling of the same hub through, and an explicit weft flag skips the check, the operator's own choice of records.
An empty weft flag then resolves the default as now, so outside a resolvable worktree a run with named slugs fails where it ran before.
`run` reads `finishedRuns(weft)` once, for `recentRuns` and for every counted slug.
After `CountRun`, a slug whose worktree `<hub>/<slug>` exists is in flight, since teardown removes it, and reads no tag; any other slug with tags calls `readArchive` with `gitrepo.New(weft)`.
The in-flight bound applies to every counted slug, named or picked by `-last`, because a reused slug's older tag makes an in-flight run look finished to `recentRuns` too: it admits any slug with an archive tag, the hazard Decision 8 bounds by the same "teardown removes the worktree" reasoning.
The report title becomes `# Token usage and time by role` (Decision 10); the output path stays.

Report.
The per-run span line names its source: with status, `Span <d> from <SpanFrom> to <SpanTo>, from the shed status at <Tag>.`; without, card 2's transcript-window line saying no shed status was read.

Tests.
`TestReadArchive`, in `count_test.go`, tier 1, is table-driven over a `fakeHistory` holding seed and status files at tag shas, a run counted from fixture transcripts, and `[]archiveTag` literals; each row asserts `SpanFrom`, `SpanTo`, `HasStatus`, `Tag`, `RoleTimes` and `Idle`, or the error:
- a seed with `started_at` starts the span there and a folded entry (`repeats` 1) ends it at its `last_at`, later than any `at`; idle is the span minus the transcripts' working time;
- a seed without `started_at` starts at the earliest transcript line, and an absent `seed.json` does the same;
- that earliest line before the next-older tag's commit time starts at the older tag's time, and the role whose turn began before it has its time clipped to the span;
- that earliest line after the first entry's end starts at the first entry's end;
- two tags read the newer one's status, whose history differs from the older's, the newer tag's `Ref` sorting before the older's so a choice by `Ref` would read the wrong one;
- a tag without `status.json`, and a `status.json` with an empty history, leave `HasStatus` false and the transcript-window span;
- a seed with an unknown key, a `status.json` that is not JSON, a history entry whose `at` is not RFC 3339 and a seed whose `started_at` is not RFC 3339 each return an error containing the slug, the tag's `Ref` and the file's relative path;
- a run slug `..` returns an error containing the slug and the tag's `Ref`.
`TestArchivedSlugs` asserts the parsed tags of each slug, `SHA` and `Committed` included, and that a line without a date is skipped; `TestRecentRuns` builds its `finished` map of tags.
`TestCalibrateGitBackedMatchesInMemory`, the package's integration test, already runs `run` end to end over its hermetic repository with the weft flag, `-hub` and `-projects`, from that repository as its working directory; it gains a phase after its calibration assertions:
- it commits a seed and a status at the paths the shedrun pair names and tags that commit `archive/alpha/<12 hex>`, and its existing `run` call's report carries the new title and alpha's span line naming the tag and the span the seed and status give;
- a second named slug, `beta`, with transcripts and no tag, keeps its role times and its span line says no shed status was read;
- with `<hub>/alpha` created, a second `run` prints alpha's span line saying no shed status was read;
- with the weft flag empty, `-hub` naming another directory returns the refusal naming the weft flag, while `-hub` spelled as the working directory's parent with a trailing slash, and as `..`, return an error that is not the refusal, since the fixture repository is no resolvable worktree.
The package's top-level tests rise by one, so its `tools/tokencount` row in `cmd/lyx/testdata/test-budget.yaml` goes from 13 to 14.

**ImpactSummary:** confined to `tools/tokencount`: `finishedRuns`, `archivedSlugs` and `recentRuns` have their callers in `run` and their own tests, every run now reads the weft's tags, and a `-hub` naming another hub needs the weft flag.

**Commit:** 3: tokencount: the run's span from its archived shed status
