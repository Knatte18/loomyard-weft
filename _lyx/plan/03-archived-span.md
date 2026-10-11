# Card 3 — archived-span

**Create:**
- `plan:tools/tokencount#ArchiveTag` -> `type ArchiveTag struct`
- `plan:tools/tokencount#RunArchive` -> `type RunArchive interface`
- `plan:tools/tokencount#weftArchive` -> `type weftArchive struct`
- `plan:tools/tokencount#newWeftArchive` -> `func newWeftArchive(dir string) weftArchive`
- `plan:tools/tokencount#weftArchive.ArchiveTags` -> `func (a weftArchive) ArchiveTags(slug string) ([]ArchiveTag, error)`
- `plan:tools/tokencount#ArchivedRun` -> `type ArchivedRun struct`
- `plan:tools/tokencount#readArchivedRun` -> `func readArchivedRun(archive RunArchive, slug string) (*ArchivedRun, error)`
- `plan:tools/tokencount#entryEnd` -> `func entryEnd(e shedengine.HistoryEntry) (time.Time, error)`
- `plan:tools/tokencount#RunTally.useArchive` -> `func (run *RunTally) useArchive(a *ArchivedRun)`
- `plan:tools/tokencount#fakeArchive` -> `type fakeArchive struct`
- `plan:tools/tokencount#fakeArchive.ArchiveTags` -> `func (f fakeArchive) ArchiveTags(slug string) ([]ArchiveTag, error)`
- `plan:tools/tokencount#fakeArchive.FileAtRevision` -> `func (f fakeArchive) FileAtRevision(rev, relPath string) ([]byte, error)`
- `plan:tools/tokencount#TestReadArchivedRun` -> `func TestReadArchivedRun(t *testing.T)`
- `plan:tools/tokencount#TestArchivedStatusFromWeft` -> `func TestArchivedStatusFromWeft(t *testing.T)`

**Edit:**
- `tools/tokencount#RunTally`
- `tools/tokencount#Report.WriteMarkdown`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/shedrun#TreeSeedRel`
- `plan:internal/shedrun#TreeStatusRel`
- `plan:internal/shedrun#DecodeSeed`
- `internal/shedengine#Status`
- `internal/shedengine#HistoryEntry`
- `internal/gitrepo#Repo.FileAtRevision`
- `internal/gitrepo#ErrPathNotAtRevision`
- `internal/gitexec#Run`
- `plan:tools/tokencount#Interval`
- `plan:tools/tokencount#RunTally.Idle`

**Intent:** Read a finished run's archived shed status and seed from its newest archive tag and derive the run's span from them (Decisions 3, 8 and 9 of the decision record).
Nothing in `run` calls this yet; card 5 wires it.

New file `tools/tokencount/archive.go`:
- `ArchiveTag` is one `archive/<slug>/<sha>` tag: its `Name` (`archive/<slug>/<sha>`), the full hex `Commit` it points at and that commit's `Committed` time.
- `RunArchive` is the read side the reader needs: `ArchiveTags(slug string) ([]ArchiveTag, error)` and `FileAtRevision(rev, relPath string) ([]byte, error)`.
- `weftArchive` embeds `*gitrepo.Repo`, whose `FileAtRevision` it inherits, and keeps the weft directory; `newWeftArchive` builds it with `gitrepo.New(dir)`.
- `weftArchive.ArchiveTags` lists `refs/tags/archive/<slug>/` with one `gitexec.Run` of `for-each-ref`, dereferencing an annotated tag to its commit, and parses each line's commit and committer date (`iso-strict`); a date that does not parse is an error naming the tag.
  `finishedRuns` keeps its own call.
- `ArchivedRun` holds the read: the chosen `Tag`; `NotBefore`, the committer time of the slug's next-older tag, zero when there is none; `Started`, the seed's parsed `started_at`, zero when the seed or the field is absent; the status's `History`; and `Ends`, each entry's `entryEnd` in recorded order.
- `entryEnd` is the entry's `LastAt` when `Repeats > 0`, else its `At`, parsed as RFC 3339.
- `readArchivedRun` takes the run's paths from `shedrun.TreeSeedRel(slug)` and `shedrun.TreeStatusRel(slug)`; a slug that fails validation is an error naming the slug.
  No tag returns nil and no error.
  Of several tags it takes the one whose commit is newest by `Committed`, ties broken by name, and `NotBefore` is the latest `Committed` strictly before it.
  It reads `status.json` at the tag's commit and decodes it into `shedengine.Status` with `encoding/json`; a status absent at the commit (`gitrepo.ErrPathNotAtRevision`) or one with no history entries returns nil and no error, the no-status fallback.
  It reads `seed.json` and decodes it with `shedrun.DecodeSeed`; an absent seed leaves `Started` zero.
  A status or seed that fails to read or decode, and an `at`, `last_at` or `started_at` that does not parse, abort with an error naming the slug, the tag and the file.

`RunTally` gains `Archive *ArchivedRun`, nil when no shed status was read, and `SpanFrom string`, the source of the span's start for the report.
`useArchive` with nil changes nothing, so the span stays the transcript window card 2 sets.
Otherwise it sets `Archive` and the span: the start is `Started`, or `run.First` when `Started` is zero; a start before a non-zero `NotBefore` becomes `NotBefore`; a zero start, or one after `Ends[0]`, becomes `Ends[0]`; the end is the latest of `Ends`.
`SpanFrom` names which of the four the start came from: the seed's `started_at`, the earliest transcript line, the next-older tag's commit, or the first shed row's finish.
`WriteMarkdown`'s per-run span line, for a run with an `Archive`, gives the span's length, its start and end as RFC 3339 UTC instants, `SpanFrom` and the tag name, in place of card 2's transcript-window line.

`TestReadArchivedRun`, in a new `archive_test.go`, drives `readArchivedRun` and `useArchive` against `fakeArchive`, an in-memory `RunArchive` keyed by slug and `<rev>:<path>` that returns `gitrepo.ErrPathNotAtRevision` for an absent file:
- of three tags, the newest by `Committed` wins although it is neither first in the slice nor last by name, and `NotBefore` is the next-older one's time;
- no tag, a status absent at the tag and a status with an empty history each return nil, and the run's span stays its transcript window;
- an undecodable status, an undecodable seed and an unparseable `at` each fail with an error containing the slug, the tag name and the file name;
- a slug `..` is refused;
- with a seed's `started_at`, the span runs from it to the latest entry end, where a folded entry ends at its `last_at` and an earlier-recorded entry finishes last;
- without `started_at` the span starts at `run.First`, raised to `NotBefore` when `First` is earlier and lowered to the first entry's end when `First` is later;
- `Idle` after `useArchive` is the span minus the union of the run's working intervals.

`TestArchivedStatusFromWeft`, in a new `archive_integration_test.go` under `//go:build integration`, builds a hermetic weft repository with `gitkit`, commits a seed and status for slug `alpha` at the paths `shedrun` names, and tags two commits `archive/alpha/<first 12 hex>` with committer dates set through `t.Setenv("GIT_COMMITTER_DATE", ...)` so the later commit carries the older date.
It asserts `ArchiveTags` returns both tags with those dates and that `readArchivedRun(newWeftArchive(dir), "alpha")` reads the status of the commit with the newer date.
`t.Setenv` makes the test process-global, so it does not call `t.Parallel`, and its comment says why.

`cmd/lyx/testdata/test-budget.yaml`: raise the `tools/tokencount` row by the two tests this card adds.

**ImpactSummary:** `RunTally` gains two fields and the per-run span line gains its archived form; with no caller of `useArchive` until card 5, the report's output is unchanged.

**Commit:** 3: tokencount: read archived shed status and seed, span from them
