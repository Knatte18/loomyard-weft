# Bolt commits skip paths neither on disk nor in git

`lyx adopt` failed at its owner write when the local, gitignored `.board-rendered.json` still named a design file that another machine had removed on Bolt and this machine had since pulled.
`renderOwnerWrite` adds every manifest name the new render no longer writes, so a file gone from both disk and git reached `git add`, which refuses a pathspec that matches nothing.
The failed write left `board.json` and `README.md` modified and uncommitted on Bolt.

`Bolt.CommitWritten` and `Bolt.PullThenCommitWritten` in `internal/fabricengine/bolt.go` now drop a returned path that is absent from disk and from HEAD's tree before staging, and commit the rest.
Every caller of the two methods gets the fix, with no change to the render manifest or to `renderOwnerWrite`'s path list.

Closes #844

## Design decisions

- The filter sits at the Bolt commit, not at the board render, because that is where paths meet git, and `boardengine` runs no git of its own (`PATTERN-fabric-git`).
- A missing path counts as tracked when HEAD's tree holds it, read through `gitrepo.Repo.FileAtRevision`, which is a local go-git read with no new `gitrepo` method.
  A missing path HEAD holds still stages its deletion, and an existing path stages as before.
  An unborn HEAD tracks nothing.
- The check reads HEAD, not the index.
  A path staged as an addition and then removed from disk before the write commits is dropped, and its index entry stays for the next whole-board sync to commit.
  This case is not expected on the board's write routes.
- `PullThenCommitWritten` skips a write's commit when nothing stageable remains, as it already did for an empty path list.
- The Bolt section of `internal/fabricengine/doc.go` and both methods' doc comments state the rule.

## Tests

The existing deleted-path subtests in `bolt_integration_test.go` and `boltpullfirst_integration_test.go` cover a path neither on disk nor in git, a missing path HEAD holds, nothing stageable, and an unborn HEAD, for both routes.

`internal/boardcli/verbs_test.go` also pins the `revised` stamps in the `list --sort age` step, so the `--sort revised` order no longer depends on which whole second each upsert landed in.

## To check by hand

No gate run is recorded in this description.
Run `lyx gate test ./internal/fabricengine ./internal/boardcli` before landing.
Reproducing the original failure on a machine with a stale manifest was not done; the tests cover the same path shape.
