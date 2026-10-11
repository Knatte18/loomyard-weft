# Card 3 — Read each run's span and producer split from its archived shed status

**Create:**
- `plan:tools/tokencount#archiveTag` -> `type archiveTag struct`
- `plan:tools/tokencount#archiveTags` -> `func archiveTags(weft string) (map[string][]archiveTag, error)`
- `plan:tools/tokencount#parseArchiveTags` -> `func parseArchiveTags(refs string) map[string][]archiveTag`
- `plan:tools/tokencount#ArchiveTrees` -> `type ArchiveTrees interface`
- `plan:tools/tokencount#ProducerTime` -> `type ProducerTime struct`
- `plan:tools/tokencount#RunTally.readShedTime` -> `func (run *RunTally) readShedTime(trees ArchiveTrees, tags []archiveTag, inFlight bool) error`
- `plan:tools/tokencount#entryEnd` -> `func entryEnd(e shedengine.HistoryEntry) (time.Time, error)`
- `plan:tools/tokencount#TestReadShedTime` -> `func TestReadShedTime(t *testing.T)`
- `tools/tokencount/shedtime_test.go`
- `tools/tokencount/shedtime_integration_test.go`
- `plan:tools/tokencount#TestShedTimeFromGitArchive` -> `func TestShedTimeFromGitArchive(t *testing.T)`

**Rename:**
- `tools/tokencount#TestArchivedSlugs` -> `plan:tools/tokencount#TestParseArchiveTags`

**Edit:**
- `tools/tokencount#RunTally`
- `tools/tokencount#CountRun`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Decisions 3, 5, 6, 8 (which tag) and 9: a run's span and its Producers split come from the shed `status.json` and `seed.json` archived at the run's `archive/<slug>/<sha>` tag in the weft repository, read through `gitrepo`'s read side and decoded through `shedrun` and `shedengine.Status`.
No shed history entry gains a start timestamp (Decision 6): each entry's finish time is all the split uses.

A new file `tools/tokencount/shedtime.go` (package `main`, file comment saying it reads a finished run's span and producer split from its archived shed records) holds every new non-test symbol above.

Tags:
- `archiveTag` holds `Name` (`archive/<slug>/<sha12>`), `SHA` (the full commit id) and `Committed` (the commit's committer date).
- `archiveTags(weft)` runs one `gitexec.Run([]string{"for-each-ref", "--format=%(refname) %(objectname) %(committerdate:unix)", "refs/tags/archive/"}, weft)` (`PATTERN-gitexec-checked-call`), keeps the error wording `reading the archive tags in %s: %w`, and returns `parseArchiveTags` of the output.
  Archive tags are lightweight (`internal/fabricengine/archive.go` runs `git tag <name> <tip>`), so `%(objectname)` is the commit and `%(committerdate:unix)` its committer date.
- `parseArchiveTags(refs)` keys each `refs/tags/archive/<slug>/<sha>` line by slug and skips a line with no `<sha>` segment, an empty slug, a ref outside `refs/tags/archive/`, or a missing or non-integer date.
- They supersede `finishedRuns` and `archivedSlugs`, which stay in place for `run` until card 5 moves `run` onto `archiveTags` and deletes them.

The run's records:
- `ArchiveTrees` is the one-method read seam `FileAtRevision(rev, relPath string) ([]byte, error)`, which `*gitrepo.Repo` and the test package's existing `fakeHistory` both satisfy.
- `RunTally` gains `Span interval` (the run's span), `StatusRead bool` (whether an archived status with history was read), `NoStatus string` (why not, empty when `StatusRead`) and `Producers []ProducerTime` (in first-seen order of producer name).
- `CountRun` ends by setting `Span` to `{First, Last}` and `NoStatus` to `No shed status was read`, so a run never passed to `readShedTime` (every run of `TestCountOrch`) already carries the transcript-window span.
- `ProducerTime` holds `Producer string`, `Entries int` and an unexported `spans []interval`, the intervals its entries own; its elapsed is `lengthOf(spans)`.
- `entryEnd(e)` parses an entry's end with `time.RFC3339Nano`: `e.LastAt` when `e.Repeats > 0`, else `e.At`.

`readShedTime(trees, tags, inFlight)` first sets `Span` to `{run.First, run.Last}`, `StatusRead` to false, `NoStatus` to `No shed status was read` and `Producers` to nil, the no-status fallback, and returns nil with that fallback when `inFlight` is true or `tags` is empty.
Otherwise:
1. The run's tag is the one with the latest `Committed`; the bound is the latest `Committed` among the other tags strictly older than it, zero when there is none.
2. `shedrun.TreeStatusRel(run.Slug)` and `shedrun.TreeSeedRel(run.Slug)` give the paths; a run id they refuse aborts with `%s: %s: %w` over slug, tag name and the error.
3. `trees.FileAtRevision(tag.SHA, statusRel)`: `gitrepo.ErrPathNotAtRevision` keeps the fallback and returns nil; any other error aborts.
   The bytes are decoded with `json.Unmarshal` into `shedengine.Status`; a decode error, or an entry whose `entryEnd` does not parse, aborts with `%s: %s: status.json: %w` over slug and tag name.
   A status with no history entries has no end to span to, so it keeps the fallback with `NoStatus` set to `The archived shed status has no history entries` and returns nil.
4. `trees.FileAtRevision(tag.SHA, seedRel)`: `gitrepo.ErrPathNotAtRevision` means no `started_at`; any other read error aborts.
   The bytes are decoded with `shedrun.DecodeSeed`; a decode error, or a non-empty `StartedAt` that does not parse as RFC 3339, aborts with `%s: %s: seed.json: %w` over slug and tag name.
5. The start is the seed's `StartedAt`, else `run.First`; it is raised to the bound when earlier, then lowered to the first history entry's end when later, and a start still zero becomes that first entry's end.
   The end is the latest `entryEnd` over the entries.
6. Each entry in recorded order owns `[prev, end)` where `prev` starts at the span start and becomes the entry's end whenever that end is later; an entry whose end is not later than `prev` owns nothing but still counts in `Entries`.
   Its interval joins its producer's `ProducerTime`, so the producers' elapsed sums to the span.
7. `StatusRead` becomes true and `NoStatus` empty.

Neither path names the `shed` segment, `seed.json` as a path, or `_lyx` (`PATTERN-shed-run-directory`, `PATTERN-lyxdirs-single-declarer`); the file names appear only in error text.

Tests:
- `TestArchivedSlugs` is renamed `TestParseArchiveTags` and rewritten over `parseArchiveTags` with the new three-field format: two tags of `run-pushes` with their SHAs and dates, one of `test-tiers`, and skipped lines for `refs/tags/archive/no-sha <sha> <date>`, `refs/tags/other/x/y <sha> <date>` and a line with no date.
- `TestReadShedTime` (new, table-driven, in `shedtime_test.go`) runs `readShedTime` over a `fakeHistory` of files keyed `<sha>:<path>`, with each row's `RunTally{Slug, First, Last}`, tags and `inFlight`, and asserts `Span`, `StatusRead`, `NoStatus`, each producer's `Entries` and `lengthOf(spans)`, or the error's text:
  - seeded: `started_at` 10:00, entries `Preflight` 10:01, `Plan-Write` 10:30, a folded `Plan-Bouncer` at 10:31 with `repeats` 2 and `last_at` 11:00, then `Webster` 10:50 (out of order, ending before the folded entry's `last_at`) and `Webster` 11:30: span 10:00–11:30, `Preflight` 1m, `Plan-Write` 29m, `Plan-Bouncer` 30m, `Webster` 2 entries 30m, and the elapsed sums to the 90m span.
  - newest tag: two tags whose files hold different statuses, the older listed first, read the newest-committed one's status.
  - no `started_at`, transcript earlier than the bound: seed `{"recipe":"loom"}`, `run.First` 08:00, the older tag committed at 09:00: span starts 09:00.
  - no `started_at`, transcript later than the first entry: `run.First` 10:05, first entry end 10:01, no seed file at the tag: span starts 10:01.
  - `started_at` earlier than the bound: `started_at` 08:00, the older tag committed at 09:00: span starts 09:00.
  - `started_at` later than the first entry: `started_at` 10:05, first entry end 10:01: span starts 10:01.
  - no start at all: `run.First` zero and no seed file: span starts at the first entry's end.
  - no tag: `Span` is `First`–`Last`, `StatusRead` false, `NoStatus` `No shed status was read`, no producers.
  - empty history: a status with `"history":[]`: the no-tag span and producers, `NoStatus` `The archived shed status has no history entries`.
  - unparseable entry end: an entry with `"at":"soon"`: the error names the slug, the tag name and `status.json`.
  - unparseable seed start: `started_at` `yesterday`: the error names the slug, the tag name and `seed.json`.
  - in flight: a tag with a readable status and `inFlight` true gives the no-tag result.
  - no status file at the tag: the no-tag result.
  - seed with an unknown key: the error names the slug, the tag name and `seed.json`.
  - status that is not JSON: the error names the slug, the tag name and `status.json`.
  - slug `a/b` with a tag: the error names `a/b`.
- `TestShedTimeFromGitArchive` (new, `//go:build integration`, in `shedtime_integration_test.go`) builds a git repository with `gitkit`, commits a status and seed under the paths `shedrun.TreeStatusRel` and `shedrun.TreeSeedRel` give for `run-x`, tags that commit `archive/run-x/<sha12>`, adds a non-archive tag, then asserts `archiveTags(dir)` returns exactly the one `run-x` tag with the commit's full SHA and the committer date `git log -1 --format=%ct` reports, and that `readShedTime` over `gitrepo.New(dir)` and that tag reads the status (`StatusRead` true, span from the seed's `started_at` to the last entry).
  It changes no working directory and calls `t.Parallel()`.
- The two new top-level tests raise the `tools/tokencount` row of `cmd/lyx/testdata/test-budget.yaml` from 12 to 14.

**ImpactSummary:** Additive: `RunTally` gains four fields nothing reads until card 4, `CountRun` sets two of them, `TestArchivedSlugs` becomes `TestParseArchiveTags` over the new parser, and the report and `run` are unchanged.

**Commit:** 3: tokencount: run span and producer split from the archived shed status
