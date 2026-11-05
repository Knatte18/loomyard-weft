# Card 1 — fabric-trailer-commits

**Create:**
- `plan:internal/fabricengine#Trailer` -> `type Trailer struct`
- `plan:internal/fabricengine#CommitAnchoredPathsWithTrailers` -> `func CommitAnchoredPathsWithTrailers(rec *Mutations, l *lymcwd.Location, relPaths []string, msg string, trailers []Trailer, opts SyncOptions) (sha string, committed bool, err error)`
- `plan:internal/fabricengine#TrailerCommit` -> `type TrailerCommit struct`
- `plan:internal/fabricengine#FindTrailerCommit` -> `func FindTrailerCommit(l *lymcwd.Location, key string, newest bool) (TrailerCommit, bool, error)`
- `plan:internal/fabricengine#AnchoredFileAt` -> `func AnchoredFileAt(l *lymcwd.Location, rev, relPath string) ([]byte, bool, error)`
- `plan:internal/fabricengine_test#TestTrailerCommits` -> `func TestTrailerCommits(t *testing.T)`

**Extends:**
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/fabricengine#CommitAnchoredPaths`
- `internal/fabricengine#appendWarpSHATrailer`
- `internal/fabricengine#Fabric.HeadSHA`
- `internal/gitrepo#Repo.Log`
- `internal/gitrepo#Repo.FileAtRevision`

**Intent:** Add the three general fabric seams D6 names, which name no drift concept (`PATTERN-fabric-isolation`) and run git only through fabric's in-process reads and its existing weft commit path (`PATTERN-fabric-git`).

- `Trailer` holds `Key` and `Value`.
  `CommitAnchoredPathsWithTrailers` commits the anchor-relative paths on the weft exactly as `CommitAnchoredPaths` does, through the same scoped pathspec and weft lock, but its message ends in a trailer block: `Warp-SHA: <the warp checkout's HEAD>` followed by one `Key: Value` line per trailer, in order.
  `CommitAnchoredPaths` writes no trailer of its own, so this seam adds the `Warp-SHA` line itself, read through `Fabric.HeadSHA`.
  A key outside the trailer key charset (`[A-Za-z][A-Za-z0-9-]*`) or a value that is empty or holds a line break is refused before anything is staged, naming the trailer; with nothing staged, `committed` is false as for `CommitAnchoredPaths`.
- `TrailerCommit` holds `SHA`, `Value` (the key's last value in the message) and `WarpSHA` (the message's `Warp-SHA` value, empty when none).
  `FindTrailerCommit` walks the weft branch's history from HEAD through `gitrepo`'s `Log` and returns the newest commit carrying the key when `newest` is true and the oldest otherwise; found is false when no commit carries it, and an unborn weft is found false.
- `AnchoredFileAt` reads an anchor-relative path at a weft commit through `FileAtRevision`; a path absent at that commit is found false, never an error.

`TestTrailerCommits` (tier1, hubforge `CopyHub`, `t.Parallel`) covers, as rows: two commits carrying `Drift-Seal`-like and `Drift-Records`-like keys, where newest and oldest each pick the right commit and value and `WarpSHA` equals the warp HEAD at commit time; a key no commit carries reads found false; `AnchoredFileAt` returns the older content at the older commit after a later commit changed the file, and found false for a path absent there; a value holding a newline is refused and the weft HEAD is unchanged; a key outside the charset, such as `Drift Seal`, is refused naming it; on a hub whose weft has no commit yet, `FindTrailerCommit` reads found false with no error.
The doc.go paragraph on weft commits gains the trailer seams.

**ImpactSummary:** Adds three fabric functions and a type; no existing caller or behavior changes.
