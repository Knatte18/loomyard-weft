# Card 3 — shed records

**Create:**
- `tools/tokencount/shed_test.go`
- `plan:tools/tokencount#ArchiveTag` -> `type ArchiveTag struct`
- `plan:tools/tokencount#ShedSource` -> `type ShedSource interface`
- `plan:tools/tokencount#ShedEntry` -> `type ShedEntry struct`
- `plan:tools/tokencount#ShedRecord` -> `type ShedRecord struct`
- `plan:tools/tokencount#ReadShedRecord` -> `func ReadShedRecord(src ShedSource, hub, slug string) (*ShedRecord, error)`
- `plan:tools/tokencount#gitShedSource` -> `type gitShedSource struct`
- `plan:tools/tokencount#gitShedSource.ArchiveTags` -> `func (g gitShedSource) ArchiveTags(slug string) ([]ArchiveTag, error)`
- `plan:tools/tokencount#gitShedSource.FileAtRevision` -> `func (g gitShedSource) FileAtRevision(rev, relPath string) ([]byte, error)`
- `plan:tools/tokencount#newGitShedSource` -> `func newGitShedSource(weft string) gitShedSource`
- `plan:tools/tokencount#fakeShedSource` -> `type fakeShedSource struct`
- `plan:tools/tokencount#fakeShedSource.ArchiveTags` -> `func (f fakeShedSource) ArchiveTags(slug string) ([]ArchiveTag, error)`
- `plan:tools/tokencount#fakeShedSource.FileAtRevision` -> `func (f fakeShedSource) FileAtRevision(rev, relPath string) ([]byte, error)`
- `plan:tools/tokencount#TestReadShedRecord` -> `func TestReadShedRecord(t *testing.T)`
- `plan:tools/tokencount#TestGitShedSourceReadsArchiveTags` -> `func TestGitShedSourceReadsArchiveTags(t *testing.T)`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** Each finished run's archived `seed.json` and `status.json` are read from the weft repository's `archive/<slug>/<sha>` tag, in a new file `tools/tokencount/shed.go`.
`ArchiveTag` holds the tag name, the tagged commit's full SHA and its committer time.
`ShedSource` is the read seam, two methods: `ArchiveTags(slug string) ([]ArchiveTag, error)` and `FileAtRevision(rev, relPath string) ([]byte, error)`.
`ShedEntry` holds one history entry's producer name and its end: `last_at` when `repeats` is above zero, else `at`.
`ShedRecord` holds the tag name, `Started` (the seed's `started_at`, zero when the seed has none), `Floor` (the committer time of the slug's next-older archive tag, zero when there is none) and `Entries []ShedEntry` in recorded order.
`ReadShedRecord` returns a nil record and no error when `<hub>/<slug>` still exists, since teardown removes the worktree and an existing one marks the run in flight, so an older run's tag for a reused slug is never read; the same nil when the slug has no archive tag, when the newest tag's tree lacks `status.json`, or when the status holds no history entry.
Otherwise it takes the tag with the newest committer time, ties broken by name, validates the slug and gets both paths from `shedrun.TreeFilesRel`, reads `status.json` and `seed.json` with `FileAtRevision` at the tag's commit, decodes the status into `shedengine.Status` and the seed with `shedrun.DecodeTreeSeed`, and parses each end as RFC 3339.
A tree lacking `seed.json` leaves `Started` zero, the case of a run archived before seeds were stamped.
A slug that fails `ValidateRunID`, a file that fails to decode and a history time that does not parse each return an error that names the slug, the tag and the file, which aborts the report; so does any read error other than a missing path.
`gitShedSource` implements the seam over the weft repository: `ArchiveTags` runs `gitexec.Run` with `for-each-ref` over `refs/tags/archive/<slug>/`, as `finishedRuns` does, and reads each tag's commit and committer time from `objectname` and `committerdate`, since fabric's archive tags are lightweight; `FileAtRevision` delegates to `gitrepo.Repo.FileAtRevision`, the read side.
The weft is only read, so no `PATTERN-fabric-git` write rule is touched.
`TestReadShedRecord` is one new top-level test in the new file `tools/tokencount/shed_test.go`, over `fakeShedSource`, declared there; the fake has the fields `tags` (a tag list), `files` (contents keyed by revision and path, a missing key answering `gitrepo.ErrPathNotAtRevision`), `readErr` (an error every `FileAtRevision` returns when set) and `asked` (the slugs whose tags were listed); the test also uses a `t.TempDir()` hub; its rows are:
- a slug with two tags reads the newest tagged commit's files and sets `Floor` to the older tag's committer time;
- a folded entry (`repeats` above zero) ends at its `last_at`;
- a seed without `started_at` leaves `Started` zero, and a tree whose `seed.json` is absent, the fake returning `gitrepo.ErrPathNotAtRevision` for a path missing from its file map, does the same;
- a `FileAtRevision` that returns any other error returns an error naming slug, tag and file, through the fake's error field;
- a status with an empty history returns a nil record;
- a seed that fails strict decoding, a `status.json` that is not JSON, a history `at` that does not parse each return an error naming slug, tag and file, and the slug `a/b` returns an error naming slug and tag, since no file is chosen before the id fails;
- a slug with no tag and a tag whose tree lacks `status.json` return a nil record;
- a slug whose `<hub>/<slug>` directory exists returns a nil record although the fake holds a tag for it, and `asked` stays empty.
`TestGitShedSourceReadsArchiveTags` is one new top-level test in `calibrate_integration_test.go`, under the `integration` tag, over a hermetic git repository: two commits whose committer dates differ, set with `t.Setenv("GIT_COMMITTER_DATE", ...)`, each tagged `archive/s/<sha>` and carrying a `status.json` and `seed.json`, and `ReadShedRecord(newGitShedSource(dir), hub, "s")` returns the newer commit's record with the older commit's date as `Floor`.
It pins the real `for-each-ref` format and the go-git blob read, which `fakeShedSource` cannot.
The two new top-level tests take the `tools/tokencount` row of `cmd/lyx/testdata/test-budget.yaml` from 13 to 15.

**ImpactSummary:** The budget file's `tools/tokencount` row is the only existing text this card edits, a number read by `TestTestBudget_PackagesStayWithinBudget` in `cmd/lyx`; the new code has no caller until card 6.
