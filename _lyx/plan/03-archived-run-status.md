# Card 3 — archived-run-status

**Edit:**
- `tools/tokencount#finishedRuns` -> `func finishedRuns(weft string) (map[string][]archiveTag, error)`
- `tools/tokencount#archivedSlugs` -> `func archivedSlugs(refs string) map[string][]archiveTag`
- `tools/tokencount#recentRuns` -> `func recentRuns(projects, hub, prime string, finished map[string][]archiveTag, last int) ([]string, error)`
- `tools/tokencount#run`
- `tools/tokencount#TestRecentRuns`
- `tools/tokencount#TestArchivedSlugs`
- `cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `plan:tools/tokencount#archiveTag` -> `type archiveTag struct`
- `plan:tools/tokencount#TreeFiles` -> `type TreeFiles interface`
- `plan:tools/tokencount#ArchivedRun` -> `type ArchivedRun struct`
- `plan:tools/tokencount#readArchivedRun` -> `func readArchivedRun(repo TreeFiles, slug string, tags []archiveTag) (*ArchivedRun, error)`
- `plan:tools/tokencount#TestReadArchivedRun` -> `func TestReadArchivedRun(t *testing.T)`

**Uses:**
- `plan:internal/shedrun#SeedTreeRel`
- `plan:internal/shedrun#StatusTreeRel`
- `plan:internal/shedrun#DecodeSeed`
- `internal/shedrun#Seed`
- `internal/shedengine#Status`
- `internal/gitrepo#Repo.FileAtRevision`
- `internal/gitexec#Run`

**Intent:** Read a finished run's archived `seed.json` and `status.json` from its `archive/<slug>/<sha>` tag in the weft repository, Decisions 8 and 9, in a new file `tools/tokencount/archive.go`.
`archiveTag` is one archive tag: `Slug`, the full `SHA` of the commit it points at, and `Committed`, that commit's committer time.
`finishedRuns` keeps its one `for-each-ref` call through `gitexec.Run` but asks for the ref name, the commit's full object name and its committer date, and returns the tags by slug, each slug's list newest committer date first; `archivedSlugs` parses that output, one tag per line, skipping a line without a slug, an object name or a parseable date.
`fabricengine` creates the tag lightweight, so the object name and date are the commit's; an annotated tag is peeled with the `*` field forms so it reads the same.
`recentRuns` keeps its behavior and takes the new map, a slug being finished when it has at least one tag; `run` passes the map through.
`TreeFiles` is the one-method read interface `FileAtRevision(rev, relPath string) ([]byte, error)`, which `gitrepo.Repo` and the existing `fakeHistory` already satisfy.
`ArchivedRun` is what was read: the `Tag` name and `SHA` chosen, the `Seed`, the `Status`, `HasStatus`, and `OlderTagCommitted`, the committer time of the slug's next-older tag, zero when there is none.
`readArchivedRun` returns `(nil, nil)` for an empty tag list; else it takes the first tag, the newest, reads the seed at `shedrun.SeedTreeRel(slug)` and decodes it with `shedrun.DecodeSeed`, reads the status at `shedrun.StatusTreeRel(slug)` and decodes it with `encoding/json` into `shedengine.Status`.
A `seed.json` absent at the tag is a seed without `started_at`; a `status.json` absent at the tag sets `HasStatus` false, the no-status fallback of Decision 3.
A slug `ValidateRunID` refuses, or a seed or status whose bytes fail to decode, is an error naming the slug, the tag and the file; the tree read's own error is wrapped the same way.
`TestReadArchivedRun`, in a new `archive_test.go`, drives `readArchivedRun` over `fakeHistory` keyed by SHA and path: a slug with two tags reads the newer one's files and carries the older one's committer time; a slug with one tag has a zero `OlderTagCommitted`; an absent seed and an absent status take the fallbacks above; a status that is not JSON, a seed with an unknown key and a slug with a slash each return an error containing the slug, the tag and the file name, which is the refusal branch these inputs reach once the tag list is non-empty.
`TestArchivedSlugs` asserts the parsed tags, their order and the skipped lines; `TestRecentRuns` passes the map shape.
One new top-level test, so the `tools/tokencount` row in `cmd/lyx/testdata/test-budget.yaml` rises by one.

**ImpactSummary:** Three re-signed functions whose callers are `run`, `TestRecentRuns` and `TestArchivedSlugs`, all in `tools/tokencount`; the tool gains an import of `internal/shedrun` and `internal/shedengine`.
