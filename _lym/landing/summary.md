# Start no pusher, and warn nothing, for a pair whose worktree is gone

Runs that had landed, and whose pairs had been removed, still had their pusher started, and each start logged a warning:

`runpusher: the pusher did not start ... resolve main worktree: list git worktrees in "<hub>/<slug>": dotgit: not a git repository: no .git above <hub>/<slug>`

The pusher start judged a pair present from its directory alone.
`TargetOf` then asked git whether the path was the prime, and the start's refusal read asked for its branches.
Both walked up from a path with no `.git` and failed.
The failure was logged once per episode for every such pair, and the next restart logged it again.

## What changes

All of it is in `internal/runpusher`:

- A pair stands only while `<worktree>/.git` exists.
  A missing path, or a directory a removal left without its `.git` entry, reads as gone.
  The pusher body already used this read, `worktreeGone`, to stop pushing a gone pair.
  Its directory-only check now also reads the leftover-directory case as gone, and it takes a path.
- `Start` and `StartIfAhead` read that first for a pair.
  A gone pair starts nothing, returns no error and reads no other condition, so it writes no ledger entry and logs no warning.
- `TargetOf` reads a worktree that no longer stands as a pair target, with no git read.
  The prime stands for as long as its hub does.
  The starts that build their target through `TargetOf` now reach the silent skip instead of failing: the dobby's run reads and the status commits.
- `TargetRefused` reads a gone pair as not refused, so the hub's refused-parent settling does not fail on one either.
- `PruneLedger` drops the entry of a worktree left without its `.git` entry, as it already dropped that of a missing path.
  An entry that holds landings is kept, as before.

## Design decisions

- **Skip at start, rather than at every caller.** Several callers start a pair's pusher, some from a ledger entry and some from a run's records.
  The start check covers all of them, including any caller added later.
- **The `.git` entry decides, not the git worktree list.** Listing worktrees needs the repository, which a removed pair's path no longer reaches.
  A stat of `.git` needs nothing else, and it tells a live worktree from the directory a removal leaves holding build output.
- **The dobby's `PairPresent` is unchanged.** It still reads the directory alone, because the dobby's leftover removal needs a leftover directory to count as present.

## Tests

- `TestStart` gains rows: a gone pair reads nothing else and spawns nothing, with or without the ahead check.
  The prime ignores the read, and a failing read is returned.
- `TestPruneLedger` gains a leftover directory without `.git`, which is dropped.
  Its standing worktree now holds a `.git` entry.
- `TestWorktreeGone_ReadsWhetherTheWorktreeStands` covers the standing worktree, the missing path and the leftover directory.
- `TestTargetOf_AWorktreeNoLongerStandingIsAPairThatStartsNoPusher` drives the real reads over both removals.
  `TargetOf` returns the pair target with no error, and `StartIfAhead` and `TargetRefused` both answer false with no error.
  On the code before this change, `TargetOf` fails with the error in the warning above.

## Merge danger

**Door:** two-way
**Blast radius:** runpusher
