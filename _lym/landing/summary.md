# fabric: fix six faults in seed, teardown, cleanup, reset and catch-up

Fixes six faults that `docs/research/hardener-works-study.md` read in the code without driving them.

## What changes

- **Seed with a refused credential.**
  A one-repository seed whose push origin refuses for authentication or permission now removes the hub
  and names the credentials as the way forward.
  It used to keep the hub and say origin may still take the push.
  `gitrepo`'s leased remote-branch update now wraps `ErrPushAuthRefused`, as the plain pushes do.
- **Empty teardown job.**
  A teardown child that gets no job keeps the slug's pending record, which an earlier teardown wrote.
- **`lym fabric cleanup --apply --remote`.**
  Before deleting a leftover branch on origin, the sweep pushes the slug's archive tags that origin lacks.
  A failed tag push keeps the branch and fills the entry's error.
  A branch whose tip's work only another local branch holds is kept with a reason.
- **`lym fabric clone --reset`.**
  It refuses a hub holding a pair's worktree, an unfinished teardown, or uncommitted, stashed or unpushed work
  in the prime code store, and names a command that moves the hub aside.
  It does not protect everything: unpushed records and board commits, uncommitted changes in the Bolt checkout
  and ignored files are still removed with the hub.
  A plain clone into an existing hub no longer advises `--reset` when the hub is whole or holds the refused work.
- **Catch-up of a run with no pair here.**
  It reads the records store's layout before fetching, through the new `fabricengine.FetchRecords`.
  A one-repository store that lost its fetch refspecs is now an error instead of a fetch of nothing.
- **`lym fabric remove <slug> --remote` on a gone pair.**
  With a pending record for the slug, it pushes the owed archive tags, drops the record
  and reports the `pending_teardown` step.
  Without one, the pair-not-found refusal stands.
  The record's way forward no longer depends on origin still holding a branch of the pair.

The `--reset` scenario in `tools/sandbox/SANDBOX-FABRIC-SUITE.md` is updated to match.
`TestStartCapture` in `driftengine` now applies its short record bound to the stalled-child rows only,
since the exited-child rows could miss it on a loaded machine.

## Design decisions

- **What `--reset` counts as work:**
  a linked worktree other than the Bolt checkout, an unfinished teardown (running ones included),
  and in the prime code store uncommitted or untracked changes, a stash, or commits origin lacks.
- **A local tag does not prove origin holds a commit.**
  When remote-tracking refs leave commits over, `--reset` asks origin what it advertises, ignoring `refs/pull/`.
  If origin cannot be read, only the tracking refs count.
- **The records store is not read by `--reset`,**
  because a seed stopped before its push leaves commits there that `--reset` has to be able to replace.
- **A hub is "whole"** when it has the Bolt checkout and a committed anchor marker in the prime's records store.

## To check by hand

- **Catch-up call site (`internal/battencli/catchup.go`).**
  Check that the no-pair fetch in `fetchRunForCatchUp` is adequately tested.
  Its test in `catchup_integration_test.go` renames the hub to get past the per-path layout cache.
- **Settle callers (`internal/fabricengine/doc.go`).**
  The text says `remove --remote` and the cancel each finish the teardown of a gone pair whose record names it.
  Check that this does not claim more than the two callers of `SettleGonePairTeardown` do.
- **Advice for a partly finished clone (`existingHubError`, `internal/fabricengine/clone.go`).**
  A clone stopped after both marks of a whole hub exist is told to work in it or move it aside, not to `--reset`.
  Check that this is right for every point a clone can stop.
- **`--reset` with local tags and pending teardowns (`hubWorkOriginLacks`).**
  Check that commits only a local tag holds, and teardowns pending or running, are all refused.
- **`--reset` with prime-store state (`hubWorkOriginLacks`).**
  Check the coverage of uncommitted or untracked files, stashes and a detached HEAD.
  Decide whether the work a reset still removes, listed under "What changes", is acceptable.
- **Failed tag push in the sweep (`cleanupremotewarp.go`).**
  Check that `TestCleanupRemoteWarp_FailedTagPushKeepsTheCandidate` covers the path that keeps the branch.
- **Merge danger.**
  Check the rating below against the sweep and the removal, which push tags to origin and, for the sweep,
  delete branches there.
- **Real credential refusal.**
  The seed test imitates GitHub's HTTP 403 with a substitute transport; try it against a real origin if possible.
- **Sandbox scenario.**
  Run the updated `--reset` steps in `SANDBOX-FABRIC-SUITE.md`.

## Merge danger

**Door:** one-way
**Blast radius:** fabric
