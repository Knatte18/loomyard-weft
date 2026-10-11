# Card 4 — Read a run's shed record from its newest archive tag

**Create:**
- `plan:tools/tokencount#archiveTag` -> `type archiveTag struct`
- `plan:tools/tokencount#RecordTree` -> `type RecordTree interface`
- `plan:tools/tokencount#readShedRecord` -> `func readShedRecord(tree RecordTree, slug string, tags []archiveTag) (ShedRecord, error)`
- `plan:tools/tokencount#TestReadShedRecord` -> `func TestReadShedRecord(t *testing.T)`

**Edit:**
- `tools/tokencount#finishedRuns` -> `func finishedRuns(weft string) (map[string][]archiveTag, error)`
- `tools/tokencount#archivedSlugs` -> `func archivedSlugs(refs string) map[string][]archiveTag`
- `tools/tokencount#run`
- `tools/tokencount#TestArchivedSlugs`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/shedrun#TreeRecordRels`
- `plan:internal/shedrun#DecodeSeed`
- `plan:tools/tokencount#ShedRecord`
- `plan:tools/tokencount#EntryEnd`

**Intent:**
Decisions 8 and 9, the reading half: list every archive tag with its commit and commit time, pick a slug's newest one, and turn the seed and status at that tag into card 2's `ShedRecord`.
Card 5 calls it from `run`.

`archiveTag` holds a tag's `Name` (`archive/<slug>/<sha>`), its commit `SHA` (full hex) and the commit's `Committed` time.
`finishedRuns` runs `gitexec.Run` with `for-each-ref --format=%(refname) %(objectname) %(committerdate:unix) refs/tags/archive/`.
Archive tags are lightweight (`fabricengine`'s `archiveWeftTip` runs `git tag <name> <tip>`), so `%(objectname)` is the commit, and `%(committerdate:unix)` is its committer time.
`archivedSlugs` parses those lines into a map from slug to that slug's tags, each slug's tags sorted newest `Committed` first, ties broken by name.
A line it cannot read is skipped, as today: one outside `refs/tags/archive/`, one with no `<slug>/<sha>` segment, one with an empty slug, one missing a field, or one whose time is not an integer.
`run` is the one caller of `finishedRuns`; it turns the map's keys into the `map[string]bool` `recentRuns` takes, and otherwise behaves as before.

`RecordTree` is the one method `readShedRecord` needs, `FileAtRevision(rev, relPath string) ([]byte, error)`; `*gitrepo.Repo` and the tests' `fakeHistory` both satisfy it.
`readShedRecord(tree, slug, tags)`, in `tools/tokencount/runtime.go`, returns a zero `ShedRecord` when `tags` is empty.
Otherwise it reads `tags[0]`, the newest tag, and sets `Floor` to `tags[1].Committed` when a next-older tag exists.
- It takes the two paths from `shedrun.TreeRecordRels(slug)`; a refused slug aborts with an error naming the slug.
- It reads the status at the tag's `SHA` with `FileAtRevision`.
  `gitrepo.ErrPathNotAtRevision` gives a record with `Read` false and `Floor` kept, the no-status case.
  Any other read error, a status that `json.Unmarshal` cannot decode into `shedengine.Status`, and a history entry whose end does not parse as `time.RFC3339Nano` each abort.
  An entry's end is its `LastAt` when `Repeats > 0`, else its `At` (Decision 3).
- `Entries` holds one `EntryEnd` per history entry in recorded order, and `Read` is true when there is at least one.
- It reads the seed at the same `SHA`; `gitrepo.ErrPathNotAtRevision` leaves `Started` zero.
  Any other read error, a seed `shedrun.DecodeSeed` refuses, and a non-empty `StartedAt` that does not parse as `time.RFC3339Nano` each abort.
  A read error other than `ErrPathNotAtRevision`, for the seed or the status, takes no row: `fakeHistory.FileAtRevision` returns only that sentinel, and the error is propagated as read.
Every abort is `fmt.Errorf("run %s: %s at tag %s: %w", slug, rel, tag.Name, err)`, naming the slug, the file and the tag (Decision 9).
The tool spells neither `_lyx`, the `shed` segment nor `seed.json`; every path comes from `TreeRecordRels`.

Tests:
- `TestArchivedSlugs` takes for-each-ref lines in the new three-field format and asserts the exact map.
  Two tags of one slug whose name order is the reverse of their time order come back newest `Committed` first; a second slug has one tag.
  The skipped lines are one with no sha segment, one outside `refs/tags/archive/`, one missing its time field and one whose time is not an integer.
- `TestReadShedRecord`, new in `tools/tokencount/runtime_test.go`, is a table over a `fakeHistory` whose file keys are built from `shedrun.TreeRecordRels`, so the test spells no shed path either.
  Each row asserts the exact `ShedRecord`, or that the error contains the slug, the tag name and the failing file's path:
  - Two tags: the newer tag's status and seed are read.
    The older tag's commit holds a different status, which must not be read.
    `Floor` is the older tag's `Committed`, `Started` the seed's `started_at`.
    One entry is folded (`repeats` 2, `at` before `last_at`) and ends at its `last_at`.
  - No tags: a zero record.
  - A tag whose tree holds no status: `Read` false, `Floor` kept.
  - A status with no seed: `Read` true, `Started` zero.
  - A seed with an unknown key: an error.
  - A seed that decodes but whose `started_at` is `yesterday`: an error naming the slug, the tag and the seed path.
  - A status that is not JSON: an error.
  - A history entry whose `at` does not parse: an error.
  - The slug `a/b` with one `archiveTag` passed, so the read gets past the empty-tags return to `TreeRecordRels`, which refuses it: an error containing `a/b`; with no path to name, this row asserts only the slug.
  It raises the `tools/tokencount` row of `cmd/lyx/testdata/test-budget.yaml` from 13 to 14.

**ImpactSummary:** `finishedRuns` and `archivedSlugs` are re-signed with `run` and `TestArchivedSlugs` as their only callers, both edited here; `readShedRecord` has no production caller until card 5, and the report is unchanged.

**Commit:** 4: tokencount: read a run's seed and status at its newest archive tag
