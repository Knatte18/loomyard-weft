# Card 3 — tokencount-shed-record

**Create:**
- `plan:tools/tokencount#ShedRecord` -> `type ShedRecord struct`
- `plan:tools/tokencount#ShedEntry` -> `type ShedEntry struct`
- `plan:tools/tokencount#RecordBlobs` -> `type RecordBlobs interface`
- `plan:tools/tokencount#shedSource` -> `type shedSource struct`
- `plan:tools/tokencount#newShedSource` -> `func newShedSource(hub, weft string) shedSource`
- `plan:tools/tokencount#shedSource.Read` -> `func (s shedSource) Read(slug string) (*ShedRecord, error)`
- `plan:tools/tokencount#archiveTag` -> `type archiveTag struct`
- `plan:tools/tokencount#parseArchiveTags` -> `func parseArchiveTags(refs string) ([]archiveTag, error)`
- `plan:tools/tokencount#entryEnd` -> `func entryEnd(e shedengine.HistoryEntry) (time.Time, error)`
- `plan:tools/tokencount#TestShedSourceRead` -> `func TestShedSourceRead(t *testing.T)`
- `plan:tools/tokencount#TestShedSourceFromWeft` -> `func TestShedSourceFromWeft(t *testing.T)`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** A finished run's shed records are read from its newest `archive/<slug>/<sha>` tag in the weft repository (Decisions 8 and 9); nothing consumes them yet.
A new file `tools/tokencount/shed.go` declares the types and functions above.
`ShedRecord` holds the tag's name `Tag`, the seed's `started_at` as `Start` (zero when the seed or its stamp is absent), `Entries []ShedEntry` (`Producer` name and `End` time, in recorded order) and `Older`, the committer date of the slug's next-older archive tag (zero when there is none).
`RecordBlobs` is the one method `FileAtRevision(rev, relPath string) ([]byte, error)`, which `*gitrepo.Repo` and the existing `fakeHistory` test fake both satisfy.
`shedSource` holds the hub directory, a `refs func(slug string) (string, error)` returning `git for-each-ref` text and a `RecordBlobs`; `newShedSource(hub, weft)` wires `refs` to `gitexec.Run` over `refs/tags/archive/<slug>/` with the format `%(refname)%09%(objectname)%09%(committerdate:iso-strict)` in the weft, and the blobs to `gitrepo.New(weft)`.
The tags are lightweight, as `fabricengine`'s archive step creates them, so `objectname` is the commit and `committerdate` its committer date; the comment says so.
`parseArchiveTags` returns the tags newest committer date first, ties by name, and fails on a line that is not three tab-separated fields with a parseable date.
`Read` returns `(nil, nil)` in each of these cases, which the caller reports as "no shed status was read": the worktree `<hub>/<slug>` exists, since teardown removes it and a present one means the run is in flight and an older run's tag for the reused slug must not be read (Decision 8's bound), checked before `refs` is called; the slug has no archive tag; the newest tag's tree has no status file (`gitrepo.ErrPathNotAtRevision`); the decoded status has no history entry.
Otherwise it takes the newest tag, sets `Older` from the second tag, and builds both paths with `shedrun.TreeFiles(slug)`; a seed absent at the tag only leaves `Start` zero.
The status is decoded with `json.Unmarshal` into `shedengine.Status`, the seed with `shedrun.DecodeSeed`, `started_at` with `time.RFC3339`, and each history entry's end by `entryEnd`: `LastAt` when `Repeats` is above zero and `LastAt` is set, else `At`, parsed as `time.RFC3339Nano`.
Every failure aborts: an error that names the slug and the tag and, for a read or decode failure, the file by `path.Base` of the `TreeFiles` path, so this file spells neither file name.
A run id that `TreeFiles` refuses has no file to name yet, so that error names the slug, the tag and the refused id.
The branches are reached by `TestShedSourceRead`, one table over `fakeHistory` blobs keyed by the paths `shedrun.TreeFiles` returns and a fake `refs`, with rows for: the newest of two tags winning and its predecessor's date landing in `Older` (the older tag's blobs are absent, so reading them would fail the row); a folded entry ending at its `last_at`, and a folded entry with an empty `last_at` ending at its `at`; a seed without `started_at`, and no seed at all, leaving `Start` zero; each of the four `(nil, nil)` cases, the in-flight row using a `t.TempDir()` hub holding the slug's directory and a `refs` that fails the test if called; and an error row each for a status that is not JSON, a seed with an unknown key, an unparseable entry `at`, an unparseable `started_at`, a `refs` line with no date and the slug `a/b`, whose `TreeFiles` refusal is reached because the fake `refs` returns a tag for it.
Each error row asserts the message holds the slug, the tag and, where there is one, the file name; the malformed `refs` line row has no tag to name and asserts the slug and the line.
`TestShedSourceFromWeft` is the only test that runs git, never runs in parallel since it sets the environment, and asserts only what the fake cannot: the prefix isolation, the real date parse and the no-tag result; it carries the `integration` build tag in its own file `shed_integration_test.go`, builds a weft repository in `t.TempDir()` with `gitkit.Git`, commits two states of one slug's records at two committer dates set with `t.Setenv("GIT_COMMITTER_DATE", ...)`, tags each as `archive/<slug>/<first 12 hex>`, adds a tag of another slug sharing a name prefix with the first, and asserts `newShedSource(hub, weft).Read` returns the newer commit's entries with `Older` the earlier date, and `(nil, nil)` for a slug with no tag; this pins the `for-each-ref` format string and the prefix against real git.
`cmd/lyx/testdata/test-budget.yaml` raises the `tools/tokencount` row by two for the two tests.

**ImpactSummary:** Only the budget file is edited; the new code is additive in `shed.go` and its callers arrive in later cards.
