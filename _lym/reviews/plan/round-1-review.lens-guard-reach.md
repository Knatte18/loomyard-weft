```yaml
lens: guard-reach
outcome: reported
```

**guard-reach: whether each guard, branch and test row the plan adds can be reached.**

Per guard, reachable or not:
- Card 1, `mergeIntoTask`'s resolver-only path with an `OutcomeStuck` result returned as the Stuck reason: reachable through the existing resolver stuck path.
- Card 1, `mergeParent` going on to `gateAndFix` after a large merge: reachable. The `TestFinalize_MergeJudgement` overlap row reaches it.
- Card 1, `TestFinalize_MergeJudgement`'s `heldAtMerge` assertion: reachable. `newTestDeps` fills `Lock` (publish_test.go:65), and the struct-literal `guard: newParentGuard(deps)` takes the lock in `Call`.
- Card 1, the new `TestFinalize_MergeInRequiredRetry` row (`p2..p3` overlap on the retry): reachable. The second pass measures from the record's `MergedTip` p2 against the new tip p3. `newTestDeps` leaves `ParentHoldsTaskWork` nil, so the overlap read runs, and the row sets `MergeBase` and `ChangedPaths` itself.
- Card 1, the per-row assertion that no `overlap-note.md` exists: the state it guards cannot occur once `overlapnote.go` is deleted, because no writer is left. **Finding.**
- Card 2, `haltReason`'s escalate end with the flag true: reachable at Finalize (`runFixer`) and at Parent-Align (`align`) on a loom run.
  - A darn run has no Parent-Align (darn-recipe.yaml defines only `Finalize` of the landing rows), so on darn only Finalize reaches it. That matches D3 and D2 and is no finding.
- Card 2, `haltReason`'s escalate end with the flag false (pawl): reachable. Pawl has Parent-Align (pawl-recipe.yaml:57) and Finalize (:148).
- Card 2, `EscalateHalt`'s Parent-Align and Finalize branches: reachable.
- Card 2, `EscalateHalt`'s `retiredMergeInName` branch, and `TestHandsBack`'s `EscalateStopPrefix` row at Merge-In: not reachable. **Finding.**
- Card 2, `restepSkips`'s escalate branch: reachable. `binaryRestep` passes a blocked or failed child status to `deps.HandBack` (innerrun_binaryrestep.go:63).
  - The failed-status negative row is reachable as an input, because failed statuses reach the seam.
- Card 2, `TestCheck_RefusesLandingHalts`'s escalate rows: reachable. `arbiterstop.Check` takes status `Error`, which carries the persisted halt reason.
- Card 3, `JudgesFirst(producer, reason)` on a stop parsed from the envelope: reachable. The step envelope's `reason` is the producer's result reason (step.go:136), and the loop copies the child's full envelope.
- Card 3, `JudgesFirst` with `Error` on a stop built from status: reachable. `recheck`'s "decide it afresh" branch (drive.go:781–797) builds the stop with `FromStatus` and the status `Error`, then decides it through `facts`.
- Card 3, the arbiter-off row: reachable through the `Judge` error path.
- Card 4, `restepHolds` (blocked state, check told): reachable from `statusCmd` and from `runLoop`'s `after` status.
  - The rows for a failed status and for no told check are reachable negatives.
- Card 4, `specFor`'s recipe branch: reachable. `c.recipe` is set in `armAt`, and the test fills the receiver by hand.
- Recurrence of the guarded site (re-running an undecided escalate stop on a new build):
  - batten's binary re-step: card 2 guards it.
  - the LLM driver's re-launch and binary watch: card 4 guards them.
  - the Go driver's hand-over and its start re-step: left unguarded on purpose, and D4's Bound accepts it.
  - No other relaunch path turned up. battenshed doc.go:130 says batten never spawns or resumes a halted child except through the binary re-step.

Findings:

1. LOW, class decision, card 2 (`plan:internal/landingshed#EscalateHalt`, `TestHandsBack`'s Merge-In row): the `retiredMergeInName` branch of `EscalateHalt` guards a state that cannot occur.
   - `EscalateStopPrefix` is new, so only this build writes it, and only from rows named Parent-Align or Finalize. No recipe row is named Merge-In any more: loom-recipe.yaml has `Parent-Align` at :382, and the constant is `retiredMergeInName = "Merge-In"` at parentalign.go:24.
   - A run an older build halted at Merge-In carries a `LandingStopPrefix` reason, never the escalate prefix (fixerrun.go:315).
   - D3 prescribes the Merge-In name, so the card follows the record. The branch and its test row still pin an unreachable input.

2. LOW, class consistency, card 1 (`TestFinalize_MergeJudgement`, "each row also asserts no `overlap-note.md` exists under `deps.ScratchDir`"): the assertion guards a state that cannot occur after the card's own deletion.
   - Card 1 deletes `overlapnote.go`, which holds `WriteOverlapNote`, `OverlapNotePath` and `overlapNoteFile`. With them gone, no code path names or writes the file (current writer: finalize.go:907–912).
   - The test therefore checks for a hard-coded filename that nothing can produce.
   - The acceptance criterion "writes no overlap note" is met by the deletion itself.

Outside my lens, one sentence: the "decide it afresh" path saves an envelope with `"reason": ""` (drive.go:782–785) as the envelope the arbiter is handed. Whether the arbiter can read the escalate halt text from that envelope belongs to the way-forward lens.

Tool calls:
1. grep of shedverbs/loop.go for `reason`/`error`: to learn what the loop envelope's `reason` carries at a blocked stop, which my context lacked.
2. grep of step.go `"reason":` and `newTestDeps`'s seams: to confirm the step envelope's reason source and which seams the retry row must set.
3. sed of drive.go:760–800, plus grep of recipe row names and `retiredMergeInName`: to trace the status-built moved-stop path and check which recipes have Parent-Align and Merge-In rows.
