# Card 11 — records-tag-reads

**Create:**
- `plan:internal/fabricengine#RecordsTag` -> `type RecordsTag struct`
- `plan:internal/fabricengine#SlugRecordsTags` -> `func SlugRecordsTags(l *lymcwd.Location, slug string, anchoredRels []string) ([]RecordsTag, error)`
- `plan:internal/fabricengine#RecordsBranchFile` -> `func RecordsBranchFile(l *lymcwd.Location, branch, anchoredRel string) (content []byte, found bool, err error)`
- `plan:internal/fabricengine#RecordsBranchHasUpstream` -> `func RecordsBranchHasUpstream(l *lymcwd.Location, branch string) (bool, error)`
- `plan:internal/fabricengine_test#TestSlugRecordsTags` -> `func TestSlugRecordsTags(t *testing.T)`

**Extends:**
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/fabricengine#ArchiveTagPrefix`
- `internal/fabricengine#setAsideTag`
- `internal/fabricengine#RecordsRepoRoot`
- `internal/fabricengine#OriginRecordRel`
- `internal/fabricengine_test#removedPair`

**Intent:** General, read-only records-repository reads that batten's D2 Land way forward needs, offered by fabric as seams any module may take (`PATTERN-fabric-isolation`), so battencli runs no git on the records repository itself (`PATTERN-fabric-git`).

- `SlugRecordsTags` lists the slug's candidate tags: its `archive/<slug>/*` tags on origin (`remoteArchiveTags`) and the records repository's local `archive/<slug>/*` and `set-aside/<slug>/*` tags.
  For each it returns a `RecordsTag` with `Name`, `Commit` (the peeled commit), `OnOrigin` (true when origin holds it), `OriginOnly` (true when only origin holds it), `CommitterTime` and `Files map[string][]byte`, the content at each of `anchoredRels` resolved under the anchor that commit itself records (its weft-root `.lym-anchor` marker, read as `recordAdoptedWeftAnchor` reads an adopted branch's), a path absent there left out of the map.
  A commit only origin holds is fetched by SHA into the records repository, writing no branch, tag or remote-tracking ref (`FetchWithArgs(origin, sha)`), before it is read.
  A records repository with no origin lists local tags only; an ls-remote or fetch failure is returned.
- `RecordsBranchFile` reads one anchored file of a records branch: from the local branch when it exists, else from origin's copy fetched by SHA; `found` false when the branch or the file is absent.
- `RecordsBranchHasUpstream` reports whether `branch.<branch>.remote` or `branch.<branch>.merge` is configured in the records repository.
- Extend `internal/fabricengine/doc.go` with the three reads.

Test `TestSlugRecordsTags`, `tier1`:
- A pair removed (`removedPair`) leaves an origin archive tag: listed with `OnOrigin` true and the origin record's content under `OriginRecordRel()`.
- The same tag deleted locally so only origin holds it: still listed, its commit fetched, `Files` filled, and no new local tag or branch.
- A `RemoveLocal` removal's `set-aside/<slug>/*` tag: listed with `OnOrigin` false.
- A requested path the commit lacks: absent from `Files`.
- The records repository's origin URL pointed at a missing path: an error naming the ls-remote.
- `RecordsBranchFile` of a local weft branch and of an origin-only one returns the origin record; of an absent branch, `found` false.
- `RecordsBranchHasUpstream` reads true for a pushed weft branch with upstream set and false after `git branch --unset-upstream`.

**ImpactSummary:** Adds three read-only exported functions and one type to fabric that no existing caller reads; no existing signature, behaviour or envelope changes.

**Commit:** `11: fabric reads a slug's records tags and a records branch's file`
