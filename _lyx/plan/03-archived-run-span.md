# Card 3 — archived-run-span

**Create:**
- `plan:tools/tokencount#ArchiveTag` -> `type ArchiveTag struct`
- `plan:tools/tokencount#SpanEntry` -> `type SpanEntry struct`
- `plan:tools/tokencount#RecordFiles` -> `type RecordFiles interface`
- `plan:tools/tokencount#ReadRunSpan` -> `func ReadRunSpan(run RunTally, hub string, tags []ArchiveTag, records RecordFiles) (RunSpan, error)`
- `plan:tools/tokencount#entryEnd` -> `func entryEnd(e shedengine.HistoryEntry) (time.Time, error)`
- `plan:tools/tokencount#ProducerTime` -> `type ProducerTime struct`
- `plan:tools/tokencount#RunTally.Producers` -> `func (run RunTally) Producers() []ProducerTime`
- `plan:tools/tokencount#writeProducers` -> `func writeProducers(w io.Writer, producers []ProducerTime, span time.Duration)`
- `plan:tools/tokencount#producerRule` -> `const producerRule = ""`
- `tools/tokencount/archive_test.go`
- `plan:tools/tokencount#TestReadRunSpan` -> `func TestReadRunSpan(t *testing.T)`
- `plan:tools/tokencount#TestProducers` -> `func TestProducers(t *testing.T)`
- `tools/tokencount/archive_integration_test.go`
- `plan:tools/tokencount#TestFinishedRunsReadsTagCommits` -> `func TestFinishedRunsReadsTagCommits(t *testing.T)`
- `plan:tools/tokencount#archiveFiles` -> `type archiveFiles map[string][]byte`
- `plan:tools/tokencount#archiveFiles.FileAtRevision` -> `func (f archiveFiles) FileAtRevision(rev, relPath string) ([]byte, error)`

**Edit:**
- `plan:tools/tokencount#RunSpan`
- `tools/tokencount#finishedRuns` -> `func finishedRuns(weft string) (map[string][]ArchiveTag, error)`
- `tools/tokencount#archivedSlugs` -> `func archivedSlugs(refs string) map[string][]ArchiveTag`
- `tools/tokencount#recentRuns` -> `func recentRuns(projects, hub, prime string, finished map[string][]ArchiveTag, last int) ([]string, error)`
- `tools/tokencount#run`
- `tools/tokencount#Report.WriteMarkdown`
- `tools/tokencount#TestRecentRuns`
- `tools/tokencount#TestArchivedSlugs`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/shedrun#TreeSeedRel`
- `plan:internal/shedrun#TreeStatusRel`
- `plan:internal/shedrun#DecodeSeed`
- `internal/shedengine#Status`
- `internal/shedengine#HistoryEntry`
- `internal/gitrepo#ErrPathNotAtRevision`

**Intent:**
Read a finished run's archived shed status and seed from its archive tag, set the run's span from them, and print the per-run and "All runs" Producers tables (Decisions 3, 5, 6, 8's tag choice and in-flight bound, and 9).
No start timestamp is added to shed history entries (Decision 6): the span and the Producers table use only the finish times history already records.
This card wires nothing new into `run` beyond the re-signed tag listing; card 4 calls `ReadRunSpan` for every counted run.

Archive tags:
- `ArchiveTag` holds `Name` (`archive/<slug>/<sha>`), `SHA` (the tagged commit) and `Committed` (its committer date).
- `finishedRuns` lists `refs/tags/archive/` with `--format=%(refname) %(objectname) %(committerdate:unix)` through `gitexec.Run`, as now, and returns `archivedSlugs`' map of each slug's tags.
  Archive tags are lightweight tags on commits (`fabricengine`'s `archiveWeftTip` runs `git tag <name> <tip>`), so `%(objectname)` is the commit and `%(committerdate)` its committer date.
- `archivedSlugs` parses each line into its slug's `ArchiveTag`s, skipping a line that is not `refs/tags/archive/<slug>/<sha>` or whose date does not parse as an integer, as it skips a malformed ref now.
- `recentRuns` treats a slug as finished when its map entry holds a tag.
- `run` passes the map from `finishedRuns` to `recentRuns` unchanged; nothing else in it changes in this card.

`RunSpan` gains `Tag string` (the archive tag read, empty when none), `Entries []SpanEntry` (each history entry's `Producer` and parsed `End`, in recorded order, nil when no status was read) and `NoStatus string` (why no status was read, empty when one was).

`ReadRunSpan` returns the run's span:
1. When `<hub>/<run.Slug>` exists as a directory, the run is in flight, since teardown removes the worktree and leaves the archive tag; it reads no tag and returns the transcript window `First`..`Last` with `NoStatus` "its worktree still exists".
   This bounds the edge Decision 8 opens: an in-flight run that reuses an older run's slug never takes that run's tag.
   The check runs for every counted run, not only a named one: a run `recentRuns` picks has an archive tag by construction, and one whose worktree still exists is the same reused-slug case, so the same bound holds for it.
2. With no tag, it returns the transcript window with `NoStatus` "no archive tag".
3. Otherwise it reads the tag whose `Committed` is newest, sorting a copy of `tags` itself, so the caller's order never matters.
   It reads `status.json` at the tag's `SHA` through `records.FileAtRevision` and `shedrun.TreeStatusRel`, and decodes it into `shedengine.Status` with `encoding/json`.
   A blob absent at the tag (`gitrepo.ErrPathNotAtRevision`) returns the window with `NoStatus` naming the tag; a status with no history entries returns the window with `NoStatus` saying the archived status records no history.
4. The span's end is the latest `entryEnd` over the history: an entry's `LastAt` when `Repeats` > 0, else its `At`, each parsed as RFC 3339.
5. The start is the seed's `StartedAt`, read through `shedrun.TreeSeedRel` and decoded with `shedrun.DecodeSeed`; a seed absent at the tag, or one with no `StartedAt`, starts at `run.First`.
   The start is then raised to the `Committed` date of the next-older tag, when the slug has one, so a reused slug's older sessions cannot set it, and lowered to the first history entry's end when it is later; a start still zero takes the first entry's end.
6. A run id `TreeStatusRel` refuses, a read error other than an absent blob, a status or seed that fails to decode, and an entry time or `StartedAt` that fails to parse each return an error naming the slug, the tag and the file.
   For the run-id refusal, the file is `status.json`, the first record it was locating; `TreeStatusRel` runs before `TreeSeedRel`, so the seed's run-id check never fires first.
   The run-id refusal is reached by a counted slug no single path segment can hold, such as `a/b`.

`RecordFiles` is the one method `FileAtRevision(rev, relPath string) ([]byte, error)`, which `*gitrepo.Repo` satisfies.

Producers (Decision 5):
- `RunTally.Producers` returns nil when `Span.Entries` is empty.
  Otherwise each entry, in recorded order, owns the interval from the latest end among the entries before it, or the span start for the first, to its own end, empty when its own end is not later, so the elapsed times sum to the span.
- Per producer name, `ProducerTime` holds `Producer`, `Entries` (history entries, a folded one counting once), `Elapsed` (summed intervals) and `Idle` (the part of those intervals `uncovered` finds outside `run.working()`).
  The slice is ordered by `Elapsed`, longest first, then by name.
- `writeProducers` writes `| producer | entries | elapsed | idle | share |`, durations rounded to the second and share as elapsed over `span`, then `producerRule`, which says a row's elapsed is the gap between its finish time and the latest earlier finish, so it includes any wait before the row started, and that `Publish` and `Finalize` record no inner breakdown.
- `Report.WriteMarkdown` writes, below each run's role table and `Span` line, a `### Producers` table for a run with entries, or, for a run without, the line "No shed status was read (<NoStatus>), so the span is the transcript window and idle covers only it.", with the reason omitted when `NoStatus` is empty.
  "All runs" gains a `### Producers` table summing `Entries`, `Elapsed` and `Idle` per producer over the runs with entries, its share over the sum of those runs' spans, printed only when at least one run has entries.

Tests:
- New top-level `TestReadRunSpan` in the new file `archive_test.go`, table-driven over `ReadRunSpan` with `hub` a fresh `t.TempDir()` and the new `archiveFiles` fake (the name `fakeRecords` is taken by `profile_test.go`), keyed `<sha>:<path>`, which returns `gitrepo.ErrPathNotAtRevision` for a missing key and a plain read error for a key present with a nil value.
  Every row past the status checks holds, at its tag, a valid status with at least one entry, so the seed rows reach the seed read.
  Each row asserts the exact `Start`, `End`, `Tag`, `Entries` and whether `NoStatus` is set, or that the error's text holds each name the row lists:
  - seed `started_at` 10:00 and entries `A` at 10:30, `B` at 11:00 folded with `repeats` 2 and `last_at` 11:40, `C` at 11:20: span 10:00–11:40, ending at the folded entry's `last_at`;
  - three tags given out of date order, each with its own status: the newest-committed tag's status is read, and a seed `started_at` earlier than the next-older tag's date starts the span at that date;
  - no seed at the tag and `run.First` 09:50 before the first entry's end: the span starts at 09:50;
  - a seed present with no `started_at` and `run.First` 09:50: the span starts at 09:50;
  - no seed and `run.First` at 08:00, before a next-older tag committed at 09:00: the span starts at 09:00;
  - no seed, a zero `run.First` and one tag: the span starts at the first entry's end;
  - a seed `started_at` 10:45 after the first entry's end at 10:30: the span starts at 10:30;
  - no tags: the transcript window, `NoStatus` set, no entries;
  - `<hub>/<slug>` created as a directory, with a tag whose status the fake holds: the window, `NoStatus` set, no entries;
  - a tag with no `status.json`: the window, `NoStatus` set;
  - a status `{"history":[]}`: the window, `NoStatus` set, no entries;
  - a status that is not JSON: an error holding the slug, the tag name and `status.json`;
  - a status read failing with a plain read error: an error holding the slug, the tag name and `status.json`;
  - an entry whose `at` is not RFC 3339: an error holding the slug, the tag name and `status.json`;
  - a seed with an unknown key: an error holding the slug, the tag name and `seed.json`;
  - a seed whose `started_at` is not RFC 3339: an error holding the slug, the tag name and `seed.json`;
  - slug `a/b` with a tag: an error holding the slug, the tag name and `status.json`.
- New top-level `TestProducers` in `archive_test.go` counts one run's transcripts with `CountRun`: a burler session working 10:05–10:15 and 10:25–10:30, and an older burler session of a reused slug working 08:00–08:30.
  It sets the run's `Span` to 10:00–11:00 with entries `Plan-Write` ending 10:20, `Webster` 10:50, `Plan-Review` 10:40 (before the latest earlier end, so empty) and `Webster` 11:00.
  It asserts `Times()` exactly: burler 15m, the older session clipped out of the archived span, idle 45m, span 1h.
  It asserts `Producers()` exactly: `Webster` 2 entries, 40m elapsed, 35m idle; `Plan-Write` 1 entry, 20m elapsed, 10m idle; `Plan-Review` 1 entry, 0s; elapsed summing to the 1h span.
  It then renders a `Report` of that run, a copy under a second slug, and a third run with no entries and a `NoStatus` reason, and asserts each run's Producers rows, the "All runs" Producers rows (`Webster` 4 entries, 1h20m0s, 66.7%), the `producerRule` text, and that the third run's section has no Producers table and carries the "No shed status was read" line with its reason.
  A last render of the third run alone asserts the report holds no `### Producers` heading.
- New top-level `TestFinishedRunsReadsTagCommits` in the new file `archive_integration_test.go`, tagged `integration`, pins the for-each-ref format against git.
  It makes a repository with `gitkit`, commits, tags the commit `archive/s/<first 12 hex>` as a lightweight tag, the way `archiveWeftTip` does, and asserts `finishedRuns` returns for `s` one tag with that name, the commit's full SHA and a `Committed` equal to the commit's `git log -1 --format=%ct`.
- `TestArchivedSlugs` feeds the new three-field lines, one with an unparseable date and one with no sha among them, and asserts the exact tags per slug.
- `TestRecentRuns` passes its finished set as a `map[string][]ArchiveTag`, one tag per finished slug, with the same expected result.
- `tools/tokencount`'s row in `cmd/lyx/testdata/test-budget.yaml` rises from 13 to 16.

**ImpactSummary:** Re-signs the archive-tag listing and `recentRuns`, whose one production caller is `run`, and adds per-run and "All runs" Producers output; no package outside `tools/tokencount` changes.
