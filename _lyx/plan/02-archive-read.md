# Card 2 — archived run records from the weft

**Create:**
- `plan:tools/tokencount#ArchiveTag` -> `type ArchiveTag struct`
- `plan:tools/tokencount#RevisionFiles` -> `type RevisionFiles interface`
- `plan:tools/tokencount#RunArchive` -> `type RunArchive struct`
- `plan:tools/tokencount#parseArchiveTags` -> `func parseArchiveTags(refs string) map[string][]ArchiveTag`
- `plan:tools/tokencount#listArchiveTags` -> `func listArchiveTags(weft string) (map[string][]ArchiveTag, error)`
- `plan:tools/tokencount#LoadArchive` -> `func LoadArchive(hub, slug string, tags []ArchiveTag, files RevisionFiles) (*RunArchive, error)`
- `plan:tools/tokencount#TestLoadArchive` -> `func TestLoadArchive(t *testing.T)`
- `plan:tools/tokencount#TestArchiveTagsGitBacked` -> `func TestArchiveTagsGitBacked(t *testing.T)`
- `tools/tokencount/archive_test.go`
- `tools/tokencount/archive_integration_test.go`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/shedrun#RunRecordRels`
- `plan:internal/shedrun#DecodeSeed`
- `tools/tokencount#fakeBase`

**Intent:** The tool reads a finished run's shed records from its `archive/<slug>/<sha>` tag in the weft repository, through `gitrepo`'s read side, and this card adds that read; nothing calls it yet.
All new production symbols go in a new file `tools/tokencount/archive.go`, package `main`, opening with a one-line file comment like `orch.go`'s.

`ArchiveTag` holds `Slug`, `Name` (`archive/<slug>/<sha>`), `Commit` (the tagged commit's full SHA) and `Committed` (its committer time).
`RevisionFiles` is the one method `FileAtRevision(rev, relPath string) ([]byte, error)`, which `*gitrepo.Repo` and the existing `fakeBase` both satisfy.
`RunArchive` holds `Tag` (the tag name read), `Seed` (`shedrun.Seed`, zero when the tag has no `seed.json`), `Status` (`shedengine.Status`) and `OlderTagAt` (the committer time of the slug's next-older tag, zero when it has one tag).

`listArchiveTags` runs `gitexec.Run` with `for-each-ref --format=%(refname)%09%(objectname)%09%(creatordate:unix) refs/tags/archive/` in the weft and passes the output to `parseArchiveTags`; a git failure is wrapped with the weft path, as `finishedRuns` wraps its own.
The format reads the tagged commit's committer time because `archiveWeftTip` makes lightweight tags with `git tag`, and the tagged object is the commit.
`parseArchiveTags` returns the tags by slug, newest `Committed` first and ties by `Name` descending, and skips a line that does not have three tab-separated fields, a ref outside `refs/tags/archive/`, a ref without a slug and a sha part, and a time that is not an integer.

`LoadArchive` returns `(nil, nil)`, which the report words as "no shed status was read", in three cases: the slug has no tag in `tags`; the worktree `<hub>/<slug>` still exists, since pair teardown removes it, so an in-flight run that reuses an older run's slug takes no older tag's status; and the newest tag's tree has no `status.json`.
Otherwise it reads the newest tag's `status.json` and `seed.json` through `files` at the tag's `Commit`, with the paths from `shedrun.RunRecordRels(slug)`, decodes the status with `encoding/json` into `shedengine.Status` and the seed with `shedrun.DecodeSeed`, and treats an absent `seed.json` (`gitrepo.ErrPathNotAtRevision`) as a zero seed.
A read error other than that, and a status or seed that does not decode, return an error naming the slug, the tag and the file (`status.json` or `seed.json`), which aborts the report.
An error from `RunRecordRels` is returned wrapped the same way; it is a fail-loud wiring guard that takes no test row, because a git ref component cannot hold a separator or be `.` or `..`, so no tag yields an invalid slug.

What this can skip: the in-flight rule skips an archive read for a slug whose worktree exists, and the bound is that the worktree's existence is the only signal; a pair's teardown removes the worktree before the report can see the run as finished.
The card adds no write and no bypass of an existing guard.

`TestLoadArchive` goes in the new `archive_test.go`, and `TestArchiveTagsGitBacked` in the new `archive_integration_test.go`.
`TestLoadArchive` is one table over the raw `for-each-ref` text, so it reaches `parseArchiveTags` and `LoadArchive` together, on `fakeBase` files keyed `<commit>:_lyx/shed/<slug>/status.json` and `.../seed.json` and a `t.TempDir()` hub.
Rows: two tags with the same commit time, where the tag whose name sorts later is read; two tags listed oldest first, where the newest tag's status is read, its seed's `started_at` is carried and `OlderTagAt` is the older tag's time; a single tag with a zero `OlderTagAt`; junk lines (a foreign ref, a line without a sha, a non-integer time) beside one valid tag; a slug with no tag; an existing `<hub>/<slug>` directory with a readable tag, which gives nil; a tag whose tree lacks `status.json`, which gives nil; a tag with `status.json` and no `seed.json`, which gives a zero seed; an undecodable `status.json`, whose error names the slug, the tag and `status.json`; a `seed.json` with an unknown key, whose error names `seed.json`.

`archive_integration_test.go` (build tag `integration`) pins the git boundary the table fakes: a hermetic repository from `gitkit` with two commits at distinct committer times set through `t.Setenv("GIT_COMMITTER_DATE", ...)`, each holding the run's `_lyx/shed/<slug>/status.json` and `seed.json` and each tagged `archive/<slug>/<first 12 hex>`; it asserts `listArchiveTags` gives the two commits' times, and `LoadArchive` over `gitrepo.New(dir)` reads the newer commit's status and the older commit's time as `OlderTagAt`.
It does not run in parallel, as it sets environment, like the existing `TestCalibrateGitBackedMatchesInMemory`.

The `tools/tokencount` row in `test-budget.yaml` goes from 12 to 14, for the two new top-level tests.

**ImpactSummary:** Only the budget row changes; every other change is a new file or a test, with no existing caller of any new symbol.
