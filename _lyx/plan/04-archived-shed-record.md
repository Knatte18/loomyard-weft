# Card 4 — archived-shed-record

**Create:**
- `tools/tokencount/shedrecord.go`
- `plan:tools/tokencount#ArchiveTag` -> `type ArchiveTag struct`
- `plan:tools/tokencount#ArchiveFiles` -> `type ArchiveFiles interface`
- `plan:tools/tokencount#readShedRecord` -> `func readShedRecord(files ArchiveFiles, hub, slug string, tags []ArchiveTag) (*ShedRecord, error)`

**Edit:**
- `tools/tokencount#finishedRuns` -> `func finishedRuns(weft string) (map[string][]ArchiveTag, error)`
- `tools/tokencount#archivedSlugs` -> `func archivedSlugs(refs string) map[string][]ArchiveTag`
- `tools/tokencount#recentRuns` -> `func recentRuns(projects, hub, prime string, finished map[string][]ArchiveTag, last int) ([]string, error)`
- `tools/tokencount#run`
- `tools/tokencount#TestArchivedSlugs`
- `tools/tokencount#TestRecentRuns`
- `plan:tools/tokencount#TestMeasureTime`

**Uses:**
- `plan:internal/shedrun#RunRecordRels`
- `plan:internal/shedrun#DecodeSeed`
- `internal/shedengine#Status`
- `internal/shedengine#HistoryEntry`
- `internal/gitrepo#ErrPathNotAtRevision`
- `internal/gitexec#Run`
- `plan:tools/tokencount#ShedRecord`
- `plan:tools/tokencount#HistoryEnd`
- `tools/tokencount#fakeHistory`

**Intent:**
Read each run's archived seed and status from its newest `archive/<slug>/<sha>` tag in the weft repository, through `gitrepo`'s read side, and turn them into the `ShedRecord` card 2 measures (Decisions 3, 8 and 9).

Archive tags:
- `ArchiveTag` is one `archive/<slug>/<sha>` tag: `Name` (the tag name without `refs/tags/`), `SHA` (the full hex commit it points at) and `Committed` (that commit's committer time).
- `finishedRuns` keeps its one `gitexec.Run` call, `for-each-ref` over `refs/tags/archive/`, with the format `%(refname)%09%(objectname)%09%(committerdate:iso-strict)`, and returns `archivedSlugs` of its output.
  Archive tags are lightweight (`fabricengine`'s `archiveWeftTip` runs `git tag <tag> <tip>`), so `objectname` is the commit and `committerdate` is its committer time.
- `archivedSlugs` parses one tab-separated line per tag into a map from slug to its tags, each slug's tags newest `Committed` first, ties by `Name`.
  A line that is not `refs/tags/archive/<slug>/<rest>` with a non-empty slug and rest, or lacks the two fields, or whose date does not parse as RFC 3339, is skipped, as the old parser skipped a malformed ref.
- `recentRuns` takes that map and treats a slug as finished when it has at least one tag; its logic is otherwise unchanged.
- `run` passes `finishedRuns`'s map to `recentRuns` as before and needs no other change in this card; the types line up through its `finished` variable.
- Update the three functions' doc comments to say what they now return.

`ArchiveFiles` is the read side `readShedRecord` needs, `FileAtRevision(rev, relPath string) ([]byte, error)`, which `*gitrepo.Repo` and the tests' `fakeHistory` both satisfy.

`readShedRecord`, in the new `shedrecord.go` (file comment: it reads a run's archived shed seed and status from the weft through `gitrepo`'s read side), returns:
- nil and no error when `<hub>/<slug>` exists, checked with `os.Stat`: the run's worktree is still there, so the run is in flight and an archive tag of its slug belongs to an older run that reused it (Decision 8's bound).
  This applies to every counted slug; a slug listed by `recentRuns` has a tag and, once torn down, no worktree, so the check only ever drops a run still in flight.
- nil and no error when `tags` is empty.
- Otherwise it reads at `tags[0]`, the newest, and sets `NotBefore` to `tags[1].Committed` when a second tag exists.
  It takes both paths from `shedrun.RunRecordRels(slug)`; that call's error is a fail-loud wiring guard, since a slug parsed out of a git tag name or used as a worktree name always passes `ValidateRunID`, and it is returned naming the slug and the tag.
  `status.json` absent at the tag (`gitrepo.ErrPathNotAtRevision`) returns nil and no error, Decision 3's no-status fallback.
  Present, it is decoded with `json.Unmarshal` into `shedengine.Status`, and each history entry's end is its `LastAt` when `Repeats` > 0, else its `At`, parsed as RFC 3339, giving `History` in recorded order.
  `seed.json` absent leaves `Started` zero; present, it is decoded with `shedrun.DecodeSeed` and a non-empty `StartedAt` parsed as RFC 3339 into `Started`.
- A read error other than an absent file, a status or seed that fails to decode, and a timestamp in either that fails to parse all return an error of the form `run <slug>: <file> at archive tag <tag>: <cause>`, `<file>` being the `RunRecordRels` path read; the caller aborts the report with it (Decision 9).

Tests:
- `TestArchivedSlugs` feeds the new three-field lines: two tags of one slug out of date order come back newest first with their full SHA and committer time, a second slug keeps its one tag, and a non-archive ref, an archive ref with no `<sha>` segment, a line missing its fields and a line with an unparseable date are each dropped; the assertion compares the whole map.
- `TestRecentRuns` passes its finished set as a map of tags; its expectation is unchanged.
- `TestMeasureTime` rows gain an optional archive input: `tags`, the files a `fakeHistory` holds keyed `<sha>:<path>` with paths from `shedrun.RunRecordRels`, a flag that creates `<hub>/<slug>`, and an expected error substring list.
  A row with archive input gets its record from `readShedRecord` instead of a literal.
  New rows, each an acceptance criterion:
  - A folded status entry with `at` 10:10, `repeats` 2 and `last_at` 10:40, under a seed with `started_at` 10:00: the span is 10:00–10:40.
  - Two tags, the newer one's status ending at 10:40 and the older one's at 11:00, the newer tag holding no seed: the span ends at 10:40, and its start is the earliest transcript line raised to the older tag's committer time.
  - A status that is not JSON, a status whose entry's `at` does not parse, and a seed with an unknown key each fail `readShedRecord` with an error naming the slug, the tag name and the file read.
  - A slug whose `<hub>/<slug>` directory exists, with a tag whose status would set the span: no record is read, `ShedTag` stays empty and the report prints the no-status sentence.
  - A slug with tags whose tree has no `status.json`: no record, the transcript window as the span.

**ImpactSummary:** the three re-signed functions are called only from `run` and from `TestArchivedSlugs` and `TestRecentRuns`, all on this card; `readShedRecord` has no production caller until card 5.
