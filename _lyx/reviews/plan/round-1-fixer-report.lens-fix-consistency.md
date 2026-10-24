```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fix's changed text agrees with its sibling cards, and look for twins left unchanged.

**LOW: card 23, new integration-steps sentence.** It says `stepNonPrimeRefusal`'s refusal "still names the task worktree", but the guard's own refusal bullet never says that refusal names the acting worktree.
- Evidence: the bullet in `_lyx/plan/23-batten-guard.md` reads "every other worktree, a task pair that is not a beam included, naming the nearest worktree up the pair's origin chain that exists here and is a beam, else the prime". It names where to run the verb and says nothing about the acting worktree.
- Today's `refuseNonPrime` names both worktrees (`internal/battencli/refusal.go:63-66`), and `stepNonPrimeRefusal` asserts `taskSlug` (`lifecycle_integration_test.go:1497`). The guard bullet and `TestResolveBattenPrime` need the same clause, or the implementer may drop the acting worktree's name and the step fails.

**LOW: card 35, new root-pair refusal before step 1.** It reads the task pair's origin record, but card 35's `**Uses:**` lists no origin reader.
- Evidence: the Uses list holds `hubCancelDeps`, `MarkBeam`, `ResetRecordsToParent`, `SpawnPairPush`, `RegenerateHubWorkspace`, `SetOwners` and `ReadTasks`, with no `internal/fabricengine#ReadOriginFor`. Card 23 lists `ReadOriginFor` for the same kind of read.
- The new `promote` refusal and the `Promoted` seam both read the record.

I found no other twins left stale:
- Card 36's `lyx fabric reconcile` sites are scoped to a missing records worktree, which matches card 2's and card 8's split.
- Card 24's Bound agrees with card 21's `Orchs` ("one `Orch` per worktree recording a strand") and with PATTERN-pair-teardown.
- Card 11's `false` refusal agrees with D3 and card 36's done-marking.
- Card 9's replay rows agree with card 11's "The replay is not routed through it".
- Card 19's records-sibling handling agrees with card 23's batten twin.

Tool calls:
- grep across `_lyx/plan/*.md` for the reworded phrases (reconcile, prime-only, reed session, root pair, ParentBranch): needed to find twins in cards my context did not show (15, 16, 25, 36).
