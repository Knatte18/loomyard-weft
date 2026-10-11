# Card 3 — archive tags and status

**Create:**
- `plan:tools/tokencount#ArchiveTag` -> `type ArchiveTag struct`
- `plan:tools/tokencount#archiveTags` -> `func archiveTags(weft string) (map[string][]ArchiveTag, error)`
- `plan:tools/tokencount#parseArchiveTags` -> `func parseArchiveTags(out string) map[string][]ArchiveTag`
- `plan:tools/tokencount#RevisionFiles` -> `type RevisionFiles interface`
- `plan:tools/tokencount#ShedEntry` -> `type ShedEntry struct`
- `plan:tools/tokencount#ShedRecord` -> `type ShedRecord struct`
- `plan:tools/tokencount#newestTag` -> `func newestTag(tags []ArchiveTag) (newest ArchiveTag, older time.Time, ok bool)`
- `plan:tools/tokencount#ReadShedRecord` -> `func ReadShedRecord(hub, slug string, tags []ArchiveTag, files RevisionFiles) (*ShedRecord, error)`
- `tools/tokencount/shedstatus_test.go`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/shedrun#ArchivedSeedPath`
- `plan:internal/shedrun#ArchivedStatusPath`
- `plan:internal/shedrun#DecodeSeed`
- `internal/shedengine#HistoryEntry`
- `internal/shedengine#Status`
- `internal/gitrepo#Repo.FileAtRevision`
- `tools/tokencount#fakeHistory`

**Intent:**
Decisions 3 (the history's ends and the seed start), 8 and 9: the tool can read each finished run's archived seed and status, and the archive tags it needs for that.
Card 6 wires these into `run` and retires the slug-only tag listing they replace.
New file `shedstatus.go` holds the new symbols.
`archiveTags` runs one `gitexec.Run` of `for-each-ref` over `refs/tags/archive/` with the format `%(refname)%09%(objectname)%09%(committerdate:unix)`, and `parseArchiveTags` turns each tab-separated line into an `ArchiveTag` (`Name` the tag name after `refs/tags/`, `Commit` the commit sha, `Committed` its committer time), keyed by the slug between `archive/` and the next `/`; a line with fewer than three fields, an empty slug or an unparsable date is skipped, and archive tags are lightweight (`archiveWeftTip` creates them with `git tag <name> <tip>`), so the tag's object is the commit.
`ReadShedRecord(hub, slug, tags, files)` returns a nil record and no error when no status is read, and the caller's section then says no shed status was read.
It returns nil when `tags` is empty; when the directory `<hub>/<slug>` still exists, since teardown removes the worktree and a present one is a run in flight, in which case no file is read and an older run's tag for the reused slug is ignored; and when the newest tag's tree holds no `status.json` (`gitrepo.ErrPathNotAtRevision`) or the status holds no history entry.
`newestTag` picks the tag whose commit has the newest committer time (ties broken by the greater tag name, so the pick is deterministic) and returns the committer time of the next-older tag, zero when there is none.
The record's fields are `Tag` (the chosen tag), `Older` (that older time), `StartedAt` (the seed's `started_at` parsed as RFC 3339, zero when the tree has no `seed.json` or the seed has none) and `Entries`, one `ShedEntry` per history entry in recorded order with its `Producer` and its `End`, which is `last_at` when the entry is folded (`repeats` > 0 and `last_at` set), else `at`.
The `status.json` path is computed and read first, then `seed.json`, so a run id the path functions reject is reported against `status.json`.
The file layout in the code is `shedstatus.go` for every symbol of this card.
Both files are read through `files.FileAtRevision(tag.Commit, path)` with the paths from `shedrun.ArchivedSeedPath` and `ArchivedStatusPath`; the seed is decoded with `shedrun.DecodeSeed` and the status with `json.Unmarshal` into `shedengine.Status`.
Every failure past the nil returns aborts: a run id the two path functions reject, a read error other than `ErrPathNotAtRevision`, a seed or status that does not decode, and a `started_at`, `at` or `last_at` that is not RFC 3339 each return an error that names the slug, the tag name and the file (`seed.json` or `status.json`).
That abort is bounded by being an error return of the report, so it can skip no data silently; the in-flight rule skips a read only where a worktree directory exists.
`shedstatus_test.go` holds two tests.
`TestParseArchiveTags`: lines for two slugs, one slug with two tags, a line lacking the date field, a ref with no sha segment after the slug (`archive/no-sha`), a line with an empty slug and a ref outside `archive/` give exactly the expected map.
`TestReadShedRecord` is one table over a `fakeHistory` whose files are keyed `<commit>:<path>`, with a temporary hub directory per row; rows: the newest of three tags is read, with the newest listed in the middle of the input and the next-older tag listed last, and `Older` is the next-older tag's date; two tags with the same commit time pick the one with the greater name; a single tag leaves `Older` zero; no tags gives nil; a slug with one tag whose commit holds a status, and whose `<hub>/<slug>` directory exists, gives nil, which shows the in-flight rule and not a missing file did it; a tree with no `status.json` gives nil; a status with an empty history gives nil; a tree with no `seed.json` leaves `StartedAt` zero and still returns the entries; a folded entry's end is its `last_at` and an unfolded one's its `at`; a status that is not JSON, a seed with an unknown key, a `started_at` that is not a time and an `at` that is not a time each return an error containing the slug, the tag name and the file name; a slug that is not a single path segment (`a/b`) returns an error containing the slug, the tag name and `status.json`.
The tokencount row in `test-budget.yaml` rises by two.

**ImpactSummary:** The only edit is the test budget row; every symbol here is new and nothing calls it until Card 6.
