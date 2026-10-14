# Card 2 — committed-file-history

**Create:**
- `plan:internal/fabricengine#CommittedAnchoredFileHistory` -> `func CommittedAnchoredFileHistory(l *lyxcwd.Location, relPath string) ([][]byte, error)`

**Edit:**
- `internal/fabricengine_test#TestCommittedAnchoredFile`

**Uses:**
- `internal/fabricengine#CommittedAnchoredFile`

**Intent:** Give D4's history check a fabric read of every committed version of one anchored file, since `PATTERN-fabric-git` rules out raw git.

`CommittedAnchoredFileHistory` lives in `internal/fabricengine/committedfile.go` beside `CommittedAnchoredFile`, its single-revision twin, and resolves the side of the pair the same way: `gitrepo.New(RecordsWorktree(l))` and the path scoped with `ScopedPathspec(l.AnchorRel, ...)` and slash-normalised.
It walks the commits that touched the path with `gitrepo`'s `PathRevisions(path, 0)`, newest first, and returns each revision's bytes read with `FileAtRevision`, skipping a revision where the path is absent (`gitrepo.ErrPathNotAtRevision`, a deletion commit), the way `StencilBaseByStamp` in `stencilhistory.go` walks its history.
A path with no history and an unborn HEAD return an empty slice and a nil error; every other failure is an error naming the path.
It is read-only, so its doc says the Fabric Git Invariant exempts it from the commit seam, as `CommittedAnchoredFile`'s does, and the file's header comment names both functions.

`TestCommittedAnchoredFile` gains a third step over the same hub, on a path of its own: a path never committed returns an empty history; after committing two versions of an anchored file through `CommitAnchoredPaths` and then editing the working copy, the history holds exactly the two committed contents, newest first, and not the edited one.
The file's header comment names the history read.

**ImpactSummary:** one new read-only accessor in fabricengine; no existing behavior changes.

**Commit:** `2: fabricengine reads every committed version of an anchored file`
