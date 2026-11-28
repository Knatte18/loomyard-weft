# Fix six fabric, cleanup and teardown faults found by reading the code

`docs/research/hardener-works-study.md` lists six faults under "Faults found by this study", all read in the code and none driven.
Each is now confirmed by a test that fails on the old code, and fixed.

## What changes

- **A refused credential during a one-repository seed.**
  A `--single-repo` clone whose seed push failed with no rejection line kept the hub and said origin may still hold the pushed refs.
  GitHub's `Permission to … denied` with an HTTP 403 prints no rejection line, so a refused credential took that path although origin took nothing.
  `gitrepo.UpdateRemoteBranchLeased` now wraps `ErrPushAuthRefused` for such a refusal, as the plain pushes already do.
  The seed reads it as a known outcome, removes the hub, and names the credentials as the way forward.
- **An empty teardown job cleared an earlier teardown's record.**
  `RunTeardownChild` removed the slug's pending-teardown record when its parent closed the input with no job.
  That parent writes no record, so the record it removed was always an earlier, unfinished teardown's.
  An empty input now leaves the record alone.
- **The cleanup sweep deleted origin's only copy of a tip.**
  `CleanupRemoteWarp` counts local tags as holding a tip's commits, but never pushed them.
  Before each delete it now pushes the slug's local archive tags that origin lacks, under the slug's teardown lock.
  A failed tag push keeps the branch and fills the entry's error.
- **`--reset` was advised and run without regard to unpushed work.**
  A clone into a whole hub, one whose Bolt checkout holds a repository, no longer names `--reset`.
  Its way forward is working in the hub, or moving it aside with `mv` before the same clone.
  A hub a killed clone left without its Bolt checkout still names the `--reset` clone.
  `--reset` now refuses a hub that holds a pair's worktree, or commits on the prime code store's branches that no remote-tracking ref or tag holds.
  The refusal names that work and the move that keeps the hub, and it comes before the clone touches the network.
- **One records fetch read no layout.**
  The catch-up's fetch of a run with no pair here ran a raw fetch of the prime's records store.
  A one-repository records store that lost its fetch refspecs then fetched nothing and reported success.
  The new `fabricengine.FetchRecords` reads the layout first, and both that fetch and `FetchPairTips` use it.
- **A `lym fabric remove` record outlived its pair.**
  A removal whose teardown child ended early leaves a record naming `lym fabric remove <slug> --remote` as its re-run.
  Once nothing of the pair was left, that command answered `pair not found`.
  With `--remote`, a removal that finds nothing of the pair now settles the slug's pending record, as the cancel already does.
  It pushes the archive tags origin lacks, drops the record, and reports the new `pending_teardown` step.
  `SettleGonePairTeardown` now reports whether it dropped a record, so the removal still refuses a slug that has none.
  A removal's way forward no longer makes its re-run conditional on origin still holding a branch of the pair.

## Design decisions

- The sweep pushes the slug's own archive tags only.
  Another local tag that holds a candidate's tip still counts as holding it and is not pushed.
- A removal settles a gone pair's record whatever the verdict on its entry, because `SettleGonePairTeardown` acts only when origin holds nothing of the pair either.
  A run owned by another machine therefore loses no branch to it.
- `--reset` does not count the records store's unpushed commits or uncommitted edits.
  A seed killed before its push leaves exactly such commits, and the `--reset` clone is that state's way forward.
- The test of `FetchRecords` drives the function over a pair's records worktree, the same code path as the prime's.
  The catch-up call site is not driven: the process caches each store's layout after the first read,
  and a hub built in the test process has already read the prime's.
- The refused-credential test points the records store's origin at an `ext::` command.
  The command prints GitHub's 403 wording, as `internal/gitrepo`'s push tests record it, and exits before speaking the protocol.

## To check by hand

- A `--single-repo` seed against a real GitHub origin whose token lacks push access was not driven.
- The sweep was driven through `CleanupRemoteWarp` with the open pull requests passed in.
  The command, which lists them through GitHub, was not.

## Merge danger

**Door:** two-way
**Blast radius:** fabric
