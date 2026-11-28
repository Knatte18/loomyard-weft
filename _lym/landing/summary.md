# runpusher: start no pusher for a worktree that no longer stands

A board sync from the prime logged one `runpusher: the pusher did not start` warning
for each task worktree that had been removed, ending in `dotgit: not a git repository`.
A removed pair can leave a directory behind at its path, and `runpusher` read that directory as a live worktree.

A worktree now stands only while it holds its `.git` entry.
A missing path, a file at the path and a directory without `.git` all read as gone.
For a pair that is gone:

- `TargetOf` returns the pair target with no error, without asking whether it is the prime.
- `Start` and `StartIfAhead` return false with no error, log nothing and write no push ledger entry.
- `TargetRefused` returns false, since nothing pushes the pair again.
- The parent seams read the pair as not standing.
- A running pusher reads its worktree as gone.
- `PruneLedger` drops its push ledger entry, unless the entry holds landings.

Closes #286

## Design decisions

- **Skip at read time.** Pair removal is unchanged.
  The start and prune paths recognise a removed pair, so directories already left behind are covered too.
- **The `.git` entry decides, not the git worktree list.**
  `fabricengine.PairWorktreeStands` answers a similar question by listing worktrees from the prime.
  The check here is one `Lstat` of `<path>/.git`, which needs no prime location and no listing.
  A directory that keeps a `.git` entry therefore reads as standing, whether or not git still registers it.
- **A gone path is typed as a pair.** The prime stands while its hub does, so `TargetOf` does not ask.
- **A failed read is still an error.** Only "not found" and "not a directory" mean gone.
  Any other failure is returned, and `PruneLedger` keeps that entry.

## To check

- Check that `## Merge danger` below does not understate what `PruneLedger` now deletes,
  or which callers see the new `TargetOf`, `TargetRefused` and `Start` answers.
- Not run by hand: a board sync on a hub that holds leftover directories of removed pairs,
  to confirm that the warnings are gone.

## Merge danger

**Door:** one-way
**Blast radius:** pushing

`PruneLedger` now also deletes the ledger entry of a path that is a file or a directory without `.git`,
when the entry holds no landings.
Any `HandOvers`, `Dropped` and `HeldRecords` in it go too, and a revert does not restore them.

The new answers of `TargetOf`, `TargetRefused` and `Start` also reach their callers in `beamcli`, `battencli` and `loomcli`.
A wrong "gone" reading would leave a live pair's commits unpushed, with no warning from the start path.
