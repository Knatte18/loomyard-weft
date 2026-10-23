# Decision record — adopt-owner-write-stale-manifest

## Goal

A Bolt write that names a path which is neither on disk nor in git, such as a design file a stale local render manifest still lists after a pull removed it, commits its other paths instead of failing `git add`.

## Scope

- `Bolt.CommitWritten` and `Bolt.PullThenCommitWritten` in `internal/fabricengine/bolt.go`: drop a returned path that is absent from disk and from HEAD's tree before staging.
- The two methods' doc comments and the Bolt section of `internal/fabricengine/doc.go`.
- Integration coverage in the existing Bolt commit tests.
- Out of scope: the render manifest's format and `renderOwnerWrite`'s path list, which keep naming every removed output.

## Decisions

### Filter at the Bolt commit, not at the board render

Decision: both Bolt commit routes drop a path that does not exist on disk and that HEAD's tree does not hold, and stage the rest as before.
Rationale: `git add` refuses a pathspec that matches nothing, so the fix belongs where the paths meet git;
every caller of the two methods gets it, and `boardengine` runs no git of its own (`PATTERN-fabric-git`).
Bound: only a path git could not stage is dropped;
a missing path HEAD holds still stages its deletion, and an existing path is staged as before.

### Tracked means held by HEAD's tree

Decision: a missing path counts as tracked when HEAD's tree holds it, read through `gitrepo.Repo.FileAtRevision`, and an unborn HEAD tracks nothing.
Rationale: the Bolt commit routes run on a board whose index matches HEAD for the paths they write, and the read goes through go-git with no new `gitrepo` method.

## Constraints

- `PATTERN-fabric-git`: git on the board runs only in `internal/fabricengine`.
- `PATTERN-gitrepo-client-boundary`: the tracked check is a local read through go-git.
- `PATTERN-test-economy`: the coverage extends the existing deleted-path subtests.

## Auto-mode assumptions

- A path staged in the index but absent from HEAD and from disk does not occur on the board's write routes, so reading HEAD alone is enough.

## Open risks

- A path staged as an addition, then removed from disk before the write commits, is dropped and its index entry stays for the next whole-board sync to commit.

## Acceptance criteria

- `PullThenCommitWritten` with a write returning a path neither on disk nor in git commits the write's other paths with no error.
- `CommitWritten` does the same.
- A missing path HEAD holds is still committed as a deletion by both routes.
