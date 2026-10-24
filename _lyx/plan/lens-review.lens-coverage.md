```yaml
lens: coverage
outcome: reported
```

Coverage lens: checking that every decision, acceptance criterion and new branch in the 37-card plan has a card and a planned row that would fail if it broke.

**MEDIUM — D4, batten's start-check owner read has no row.**
- The criterion says "batten's start check reads an unspawned beam member as owned by the beam's owner".
- Card 15 swaps `internal/battencli#hubOwnerSeams` to `hubgeom.ParentOf`, but its only rows are `TestParentOf` and `TestHubSeamOptions`, which are teardown seams.
- No row drives `refuseStartOwnedElsewhere` over a beam member with no owner recorded.

**MEDIUM — D4, nothing pins that the origin record wins over the board.**
- The criterion: on the machine holding its pair, a member spawned under a grouping project that later became a beam reads `parent_beam` as the prime.
- `TestParentOf`'s row "a pair forked from the prime reads `""`" (card 15) writes no board naming a beam for that slug.
- A board-first implementation would still pass it.
- Needed: a row where the origin names the prime and the board's `ParentBeam` names a beam.

**MEDIUM — D5, the dirty-beam landing criterion has no row.**
- The criterion: a member landing into a dirty beam is Stuck at Finalize naming commit or stash in the beam, and resumes once the beam is clean.
- No card plans a test for it. Card 4 tests only the fork side.

**MEDIUM — D6, the weft-side checked-out refusal is unreached.**
- Card 5's Intent refuses a records branch checked out in another records worktree, naming that worktree.
- `TestAdd_AdoptLocal`'s rows cover only the warp branch checked out elsewhere.

**MEDIUM — D8, no per-verb row for running from a beam.**
- The criterion names `lyx batten run`, `status` and `cancel`, `lyx adopt`, `lyx cleanup` and the `lyx beam` verbs run from a beam acting on the prime.
- Card 23 tests only `TestResolveBattenPrime`, plus self-pair refusal rows.
- No row runs any of those verbs from a beam, so a call site left on `RefuseUnlessPrime` passes every planned test.

**MEDIUM — D8, orch-in-beam state location and strand name are untested.**
- The criterion: `lyx orch start` runs in a beam "with its state under the beam and its strand `<shortname>:<slug>:orch`".
- Card 19's rows (`TestRefuseNonPrime`, `TestRefuseBeamStart`, `TestOrchLiveFrom`) assert neither.

**MEDIUM — D10, re-runs after steps 1 and 5 are unreached.**
- The criterion: "a re-run after any step finishes it".
- Card 35's `TestPromote` finishes from a stop after steps 2, 3 and 4 only.
- The finish-step-6 path ("a project whose `beam` is true and whose pair carries the mark finishes step 6") is in the Intent but has no row.

**MEDIUM — D11, grouping-project depth in the open-member refusal is untested.**
- The criterion: `land` is refused with an open member "at any depth".
- Card 36's `TestBeamLandRefusal` judges told facts through `beamLandRefusal`.
- The depth resolution lives in the reader `BeamLandRefusal` (through `hubgeom.ParentOf`), which no row drives.

**MEDIUM — D11, the after-claim re-check in Worktree-Create has no row.**
- Card 36's Intent: "Worktree-Create applies it again after its claim … a member added before the claim is caught."
- No row reaches the post-claim refusal; the rows are `TestStartGuard_Refuse` and `TestLand`.

**MEDIUM — D11, the end-to-end landing has no row.**
- The criterion: the landing merges the beam into its parent, tears it down, leaves no orch, and the teardown settle marks the project done.
- Card 29 covers only the no-reroute merge judgement.
- No row shows the teardown settle marking a project entry done.

**MEDIUM — D12, the sub-beam case is untested.**
- The criterion: a beam file lists its own level only, "a sub-beam by its own worktree only".
- Card 7's `TestRegenerateHubWorkspace` rows have no nested beam.
- They also cover the "gone" parent but not an "unrecorded" one (no origin record, or an empty `ParentWorktree`).

**MEDIUM — D13, `noticesReachDriversParent` and `runParentRecorded` have no row.**
- Card 25 rewrites both, but its only row is `TestNoticeSite`.
- The record names `noticesReachDriversParent` among the readers that take the walk.

**MEDIUM — D14, the slug refusal's way forward is not asserted.**
- Card 13's `TestSlugRule` rows assert only the reason text.
- The Intent's admitted-slug way forward (suffix dropped, or `-task` appended) is never checked.
- PATTERN-refusal-way-forward requires a test that reaches the way forward.

**MEDIUM — Card 8, two refusal branches are unreached.**
- `ResetRecordsToParent` refuses a missing origin record (naming `lyx fabric reconcile`) and a root pair with an empty `ParentBranch`.
- `TestResetRecordsToParent` reaches neither.

**LOW — smaller untested branches.**
- **D6:** "batten's `Add` refuses `ErrBranchExists`" is argued through the fabric row only.
- **Card 6:** `--local --adopt-local` refused is unreached; only `--local --parent` has a row.
- **D7:** "whichever worktree starts it" has no row starting from a beam.
- **D9:** "a member started mid-create … later resumes into the beam" has no row; the create envelope's `lyx orch start` pointer is unasserted.
- **D12:** "a prime spawn writes the same files" has no row.
- **D13:** "hibernate's and the push notifier's notices reach the prime's orch" has no row.
- **D14:** "`lyx beam promote` checks none" has no row. The board-level junction-name case is untested, since `TestSlugRule` and `stepProjects` use only the records suffix.
- **Card 10:** the merge's `order` rewrite to the upserted slug has no row.
- **Card 11:** `checkBeamPayload` on the batch and merge paths has no row.
- **Card 12:** the resolver error leaving `parent_beam` empty, and the cycle stop, have no row.
- **Card 20:** `ParentOrchRecorded` has no row.
- **Card 26:** `NotifyOrchAt`'s no-strand-recorded no-op has no row.
- **Card 28:** a `branchOnOrigin` error has no row.
- **Card 32:** `TestWire_BeamRecipe` does not assert that `RerouteReviewed` is nil, which is what makes a spent or over-cap fixer halt.
- **Card 33:** `childSeedParams`' beam `parent` param and the three-recipe `unsupportedChildRecipeRefusal` text have no row.

No further tool calls were made.
