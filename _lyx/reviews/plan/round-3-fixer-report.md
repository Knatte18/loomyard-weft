# Round 3 fixer report — plan

## Changes

- **F1** (fixed as proposed): `_lyx/plan/12-landing-merge-in-producer.md` step 3 now has `syncParent` return `FastForwardResult.NewHead` after a fast-forward, and notes it can be newer than the `Upstream` that `ReadParentTips(true)` read.
  The `mergein_test.go` behind row now has the fake puller return a `NewHead` that differs from the fake upstream and asserts the merge record's `merged_tip` equals that `NewHead`.
- **F2** (fixed differently): `_lyx/plan/07-fabric-parent-reads.md`'s `FastForwardPair` bullet now states `Pull`'s weft guards.
  A weft with no upstream is skipped, a `PullWeft` failure is a Warn that does not stop the warp, a moved weft HEAD sampled with `weftSHAOrEmpty` appends `KindRepoAdvanced` at the weft path, and `opts.SkipGit` skips the weft side.
  `Fabric.weftHasUpstream` and `weftSHAOrEmpty` were added to Uses.
  The test paragraph gains a case where a parent pair whose weft has no upstream fast-forwards the warp with a nil error.
  It also asserts the weft advance entry when the weft's upstream moved, because the review's issue names the unrecorded weft advance as a gap the test misses.
  Not done as proposed: the fix said "pulls the weft first".
  On a dirty worktree or a not-fast-forward upstream, that would move the weft before the refusal.
  The card promises each refusal with "nothing moved" and an empty record, and its test asserts "the pair unmoved and an empty record".
  So the weft pull runs once no refusal applies, before the warp reset.
  That keeps the refusal contract, and the weft side still runs before the warp moves, as in `Pull`.
- **F3** (fixed as proposed): `_lyx/plan/22-status-lock-wait.md`'s spec paragraph now reads "which a landing row writes while it waits for the parent lock".

## Disputed

None.

## Deferred

None.
