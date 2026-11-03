# Commit a task's clear of inherited friction notes with its first seed

Fixes [issue #22](https://github.com/Knatte18/loomyard/issues/22).

## Problem

A task worktree branches from its parent's weft, and that weft can track friction notes.
A beam's pawl run, for one, commits its own `loom-halt.md` with its run records.
On a task's first seed, `ensureFrictionDirAfterSeed` clears the friction directory so the task does not reflect on notes it did not write.
The bootstrap commit that followed staged only the status file, the seed and the origin record, so the deletion of the inherited note stayed uncommitted.
Preflight's worktree-clean check then refused the run on that deletion, and every task run released in such a beam halted blocked at Preflight.

## Change

`seedAndCommitBootstrap` (`internal/loomcli/sharedbootstrap.go`) now adds the friction directory to the bootstrap commit whenever Tier 2 is on and the directory holds tracked or untracked content.
The first seed's clear is then committed together with the seed, and the worktree is clean when Preflight checks it.

- `bootstrapCommitPaths` takes a `withFriction` flag and appends `loomengine.LoomFrictionDirRel()` when it is set.
- `fabricengine.AnchoredPathHasContent` is a new read beside `CommitAnchoredPaths`.
  It reports whether an anchor-relative path matches anything the run-records worktree's index tracks, or an untracked file that is not ignored.
  It reuses `entryMatchesWeft`, the probe the weft pathspec filter already uses.
  The bootstrap asks it first because `git commit -- <dir>` refuses a pathspec that matches nothing, which is the case for the empty, never-tracked friction directory of an ordinary fresh task.
  A tracked note deleted from disk still matches, so its deletion is staged and committed.
- The probe runs on every bootstrap, not only on a first seed.
  This keeps the self-healing the other bootstrap paths already have: if a first seed's clear landed on disk but its commit crashed, the next `start` or `step` commits it.
  On a resume, notes the run has written so far are committed with the bootstrap commit.
  They are run records that `commit-records` commits anyway, so this only commits them earlier.

## Design decisions

- The issue offered two fixes: stop committing a run's halt note into the weft a child branches from, or have the child commit its own clear before Preflight.
  This change takes the second.
  Halt notes are run records, and the pawl run needs its note committed and pushed like any other record.
  The child already clears inherited notes on purpose, so committing that clear removes the dirty tree without changing what the parent records.
- With Tier 2 off nothing is cleared, so an inherited note stays tracked and unchanged, the tree stays clean and no friction path is staged.

## Tests

- `TestSmokeStepBootstrapWiring` (tier 2) now commits the stale friction note into the weft before the first `lym loom step`, which reproduces the inherited-halt case.
  It asserts that the note is cleared and that the run is neither blocked nor failed after Preflight.
- `TestBootstrapCommitPaths` replaces `TestBootstrapCommitPaths_IncludesSeedRel` and covers the path list both with and without the friction directory.
- The other bootstrap smoke tests run fresh tasks whose friction directory is empty and untracked.
  They pass, which shows the bootstrap does not stage that empty directory.

## Docs

The `# Start` section of `internal/loomcli/doc.go` and the friction-notes paragraph in `docs/overview.md` now say that a first seed commits its clear of the friction directory.
