# Bolt writes: leave out a path neither on disk nor in HEAD

Fixes #844.

An owner write, such as the one `lyx adopt` makes, failed at `git add` with `pathspec ... did not match any files` when this machine's local `.board-rendered.json` was older than a pull.
`renderOwnerWrite` returns every name the previous render's manifest lists and the new render no longer writes, so a design file another machine had already removed on Bolt reached `git add`, which refuses a path matching nothing.
The failed write also left `board.json` and `README.md` modified and uncommitted.

## What changes

`Bolt.CommitWritten` and `Bolt.PullThenCommitWritten` in `internal/fabricengine/bolt.go` now pass a write's paths through `stageablePaths` before staging.
A path on disk is staged as before.
A missing path HEAD's tree holds is staged as before, which commits its deletion.
A missing path HEAD does not hold is left out, and an unborn HEAD holds no path.
When nothing stageable is left, the write commits nothing.

## Design decisions

- The filter sits in the Bolt commit routes rather than in `renderOwnerWrite`, because that is where the paths meet git: every caller of the two methods gets it, and `boardengine` runs no git of its own.
  Checking the manifest names against the disk inside `renderOwnerWrite` would still fail on a rendered file that was never committed.
- "Tracked" means held by HEAD's tree, read through the existing `gitrepo.Repo.FileAtRevision`, so no new `gitrepo` read is added.
  A path staged in the index but absent from both HEAD and disk is left out too, and its index entry waits for the next whole-board sync.

## Docs

The two methods' doc comments and the pull-first paragraph of `internal/fabricengine/doc.go` state the new staging rule.

## Tests

The deleted-path subtests of `TestBolt_CommitWritten` and `TestBolt_PullThenCommitWritten` now also return a path that is neither on disk nor in HEAD, and assert that the commit lands with exactly the other changes.
