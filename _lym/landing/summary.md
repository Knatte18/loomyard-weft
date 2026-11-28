# Verify with no lock when the parent is ahead only by owed landings

With local landing, a parent branch can be ahead of origin by earlier landing tips whose push is still owed.
Finalize's lock-free pass merged the upstream tip of such a parent, while the locked pass compares that tip with the parent's local head.
The two never matched, so each landing in that state lost all of its `landing.finalize_lost_races` races
and then merged and ran verify while holding the parent lock, making other landings on the same parent wait.

The lock-free pass now merges and verifies the local tip of a parent that is ahead by owed landing tips alone.
The landing then goes through under the lock unless another landing has moved the parent.

Closes #292

## Design decisions

- `parentMergeTip` takes a `takeOwedLocal` flag.
  Only Finalize's lock-free pass sets it; Parent-Align still merges the upstream tip of an owed-ahead parent.
- When `Deps.ParentAheadByOwed` first answers true, which is read with no lock,
  it is asked again under the lock taken to read the local tip.
  If a commit that is no owed landing has arrived in between, the upstream tip is merged.
- A parent ahead by anything else, or an unwired `ParentAheadByOwed`, still gets the upstream tip.
  A failed read of `ParentAheadByOwed` is returned as an error.
- A landing known from local reads to need GitHub merges the upstream tip,
  since it cannot land until origin holds the owed tips.
  A failed read of that need also keeps the owed tips out.
  `needsGitHubOnDisk` therefore returns its read error instead of folding it into `false`.
- Known limit: a landing that needs GitHub only because of GitHub's repository rules is not recognised in the lock-free pass,
  since reading the rules is a GitHub call.
  On an owed-ahead parent, this change makes that pass merge the owed tips into the task worktree for such a landing.

## To check by hand

- Check that `TestFinalize_LockFreePassMergesTheUpstreamTipOfAnAheadParent`
  in `internal/landingshed/finalize_unlocked_test.go` covers the lock-free pass's fallbacks to the upstream tip
  for an ahead parent.
- Check that the owed-only answer is read again under the parent lock,
  in `parentMergeTip` (`internal/landingshed/parentalign.go`).
- Check that the owed-ahead case of `TestMergeIn_MergeTip` in `internal/landingshed/parentalign_test.go`
  pins Parent-Align to the upstream tip of an owed-ahead parent.

## Merge danger

**Door:** two-way
**Blast radius:** landing
