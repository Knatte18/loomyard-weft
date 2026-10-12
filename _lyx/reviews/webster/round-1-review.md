---
verdict: APPROVED
findings:
  - id: F1
    severity: NIT
    class: consistency
    location: internal/landingshed/publish.go:196
    summary: Two Publish comments still describe a merge-in that Publish no longer runs
    fix: In internal/landingshed/publish.go, at line 196 delete the words "and no second merge-in" so the comment reads "No second pull request created."; at line 96 change "with no merge-in, no push, and no GitHub call whatsoever" to "with no lock, no push, and no GitHub call whatsoever".
---

### [NIT] Stale merge-in wording in Publish comments

**Location:** internal/landingshed/publish.go:96 and internal/landingshed/publish.go:196
**Issue:** Card 18 moved the merge-in out of Publish into Merge-In, and card 20 made Publish take the parent lock.
Two comments in `Publish.Call` still speak of a merge-in Publish performs or skips: the step 2 comment says it returns Done "with no merge-in", and the open-pull-request case says "no second merge-in".
Neither is false in effect, but they name a step Publish no longer has, which misleads a reader of the row.

## Verdict basis

The range is `git diff $(git merge-base main HEAD)..HEAD`: `_lyx/shed/merge-review/status.json` names `main` as `product.parent`, and no `_lyx/loom/rework/` directory exists.
No Publish failure is on record.
`lyx gate test` over parentlock, shedengine, shedverbs, shedbuild, shedrecipe, landingshed, loomcli, battenshed and battencli passed.
The focus directive's items, each judged against the code:

- **Lock identity and release.**
  `Store.Release` and `Store.HandOff` compare `SameProcess`.
  `parentGuard.release`, the loop's exit and wake hooks, and batten's `Release` (which passes the record it read, owner included) all go through them, so a newer owner's record is not freed.
  Only the operator's `lock release` verb forces, by design.
- **Lock Stuck halting.**
  `ensure` returns a reason for `ErrPaused` and `ErrUnreadable`.
  `holdParent` and Merge-In return it through `stuckVerdict(..., halt=true)`.
  The engine blocks a `Halt` Stuck at its own row ahead of the `OnStuck` branch and exempts it from the bounce budget.
  PR-Gate therefore halts at its row instead of routing to PR-Rework.
- **Moved-parent test.**
  `parentGuard.moved` treats a missing or unreadable merge record as moved.
  It compares both tips with `merged_tip` and applies `HoldsWork`.
  It fetches only when the push is not skipped and ignores the upstream tip when the push is skipped or the branch has none.
  This matches the plan.
  Batten's `parentMovedSinceMergeIn` reads upstream only and treats a missing record as not moved, which card 24 specifies for the backstop.
- **Reroute edge.**
  `rerouteCount` counts back to the row's latest Done or reroute-cap halt.
  The fourth reroute blocks with `RerouteCapped`, the reroute entry ends the target segment's episode and spends no budget, and a row with no `OnReroute` is rejected by `validate` and fails the run at step time.
  `on_reroute` is parsed in `shedbuild` and appears on Publish, PR-Gate and Finalize in both recipes, and all three reach `moved` only through `holdParent` or `rerouteIfMoved`.
- **Recipe flip.**
  Merge-In and the Merge-Review perch sit between Describe and Publish.
  Darn goes straight from Describe to Merge-In to Publish.
  `skip_merge_review` routes Merge-In to Publish, and Publish carries no merge-in.
  Card 18's rename and deletions are applied: `TestSequence_FullRunBlocksAtMergeIn` exists, and `TestPublishVerify_ResolverStuck` and `TestPublish_CommitStatus` are gone.
  Card 20's `Publish.noteMergeCommit` deletion is applied.
- **Run records stay ephemeral.**
  The merge record, reviewed-tip file and wait record are all built from `loomengine.LoomScratchDir`, which sits under the `.lyx` directory name.
- **Told geometry.**
  `parentlock.Dir` is built only in `loomcli` and `battencli` wiring, and `parentlock` is in the told-geometry bound list and its seam test passes.
- **Test economy.**
  I found no duplicated test and no uncovered added behavior.
  Reroute, halt, lock identity, guard, rows, loop hooks, batten backstop and lock verbs each have a test, and the fabric parent reads have an integration test.
