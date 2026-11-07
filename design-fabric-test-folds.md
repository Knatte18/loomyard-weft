# Fold the overlapping internal/fabricengine tests the tier 1 report read

`fabric-test-folds` · Task · enhancement, fabric, testkit, git · running · Darn-Write

Filed from the Sketch `fabric-test-speed`.

Takes: today's fixtures and the report's section 4 rows with their alone-pins column
Delivers: TestVerbCases_CleanState's preconditions in runCell, and the TestAnchorChange, TestStoreHousekeeping, TestReconcileOrigin_Moves, TestPushFailureClasses and TestBolt_PullThenCommitWritten folds, each commit naming the assertions its folded row alone made
Exports: nothing

The part changes these packages:

- internal/fabricengine (about 500 changed lines)
