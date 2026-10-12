# Card 4 — tokencount-shed-records

**Create:**
- `tools/tokencount/records.go`
- `plan:tools/tokencount#RunArchive` -> `type RunArchive interface`
- `plan:tools/tokencount#ShedRecords` -> `type ShedRecords struct`
- `plan:tools/tokencount#archiveTagPrefix` -> `const archiveTagPrefix = "archive/"`
- `plan:tools/tokencount#readShedRecords` -> `func readShedRecords(archive RunArchive, hub, slug string) (*ShedRecords, error)`
- `tools/tokencount/records_test.go`
- `plan:tools/tokencount#fakeArchive` -> `type fakeArchive struct`
- `plan:tools/tokencount#fakeArchive.TagsUnder` -> `func (f fakeArchive) TagsUnder(prefix string) ([]gitrepo.TagRef, error)`
- `plan:tools/tokencount#fakeArchive.FileAtRevision` -> `func (f fakeArchive) FileAtRevision(rev, relPath string) ([]byte, error)`
- `plan:tools/tokencount#TestReadShedRecords` -> `func TestReadShedRecords(t *testing.T)`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/gitrepo#TagRef`
- `plan:internal/gitrepo#Repo.TagsUnder`
- `internal/gitrepo#Repo.FileAtRevision`
- `internal/gitrepo#ErrPathNotAtRevision`
- `plan:internal/shedrun#ArchivedRels`
- `plan:internal/shedrun#DecodeSeed`
- `internal/shedengine#Status`

**Intent:** Read a finished run's archived `seed.json` and `status.json` from the weft repository, per Decisions 8 and 9 of the decision record.
`RunArchive` is the read side the tool needs, `TagsUnder` and `FileAtRevision`, which `*gitrepo.Repo` satisfies and the test fakes in memory, the way `PlanHistory` and `RunRecords` already do.
`ShedRecords` holds the chosen tag's name, the decoded `shedrun.Seed`, the decoded `shedengine.Status`, and `PriorTagAt`, the committer time of the slug's next-older archive tag, zero when the slug has one tag.
`readShedRecords(archive, hub, slug)` applies the in-flight bound first: when the directory `<hub>/<slug>` exists it returns nil records and no error, since teardown removes the worktree and a run whose worktree stands is still in flight, whatever older tags the slug has.
It then lists the tags under `archiveTagPrefix + slug + "/"`, returns nil records when there are none, and picks the tag whose commit is newest by committer time, with the tag name as the tie-break, taking the next-newer committer time as `PriorTagAt`.
It takes the two paths from `shedrun.ArchivedRels(slug)`, reads `status.json` at the tag's commit with `FileAtRevision` and decodes it into `shedengine.Status` with `encoding/json`, and reads `seed.json` and decodes it with `shedrun.DecodeSeed`.
An `ErrPathNotAtRevision` for `status.json` is nil records, since a pair torn down before its run was seeded holds none, and an `ErrPathNotAtRevision` for `seed.json` alone leaves the zero `Seed`, so the span start falls back as Decision 3 says.
A run id `ArchivedRels` refuses, any other read error, and a seed or status that fails to decode each abort with an error naming the slug, the tag and the file, which `run` returns unchanged.
The tool spells no `_lyx`, `shed` or `seed.json` literal: every path comes from `shedrun`.
`records.go` holds the above with a header comment saying it reads a run's archived shed records through gitrepo's read side.
`TestReadShedRecords` in the new `records_test.go` drives `readShedRecords` over a `fakeArchive`, tags with commit SHAs and committer times plus file contents keyed by `<sha>:<path>` like `fakeHistory`, and a `t.TempDir()` hub, as a table: a slug with three tags reads the newest tagged commit's status and reports the next-older tag's time as `PriorTagAt`;
a slug with one tag has a zero `PriorTagAt`;
a slug with no tag returns nil;
a slug whose `<hub>/<slug>` directory exists returns nil although a tag exists;
a status that is not JSON, and a seed with an unknown key, each fail with an error naming the slug, the tag and the file;
a tag with no `status.json` returns nil;
a tag with a status and no seed returns records with a zero seed;
and a slug holding a path separator fails naming the slug.
Raise the `tools/tokencount` row of `cmd/lyx/testdata/test-budget.yaml` from 13 to 14 for the new test.

**ImpactSummary:** New file and test only; nothing calls readShedRecords until card 7 wires it, so no report output changes here.
