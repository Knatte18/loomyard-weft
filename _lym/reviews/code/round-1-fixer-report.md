# Fixer report, round 1

## Changes

- F1: `TestCompact_HookMarkStampedBeforeTheBoundaryIsNotTypedAgain` (now `TestCompact_HookDeliveredPointerIsSkippedAndAStaleMarkIsTyped`) is a two-row table; the second row sets `CompactionBaseline` before the cycle and writes a same-text mark one second older, expecting the pointer typed. Fixed as proposed.
- F2: `TestWatcher_ClearCycleTypesThePointerWhenTheMarkPredatesTheBaseline` in `watcher_test.go` drives a clear cycle with a set baseline and an older same-text mark; expects the pointer typed and `PointerMarkSince` zero after idle. Fixed as proposed.
- F3: `TestResetForFreshLaunch` non-idle input sets `PointerMarkSince`, and the test asserts it is zero. Fixed as proposed, no new test function.

Mutation check: deleting `watcher.go:1233` and `watcher.go:1649` made the F2 test and the F1 second row fail; the lines were restored. The `orchengine` tier0 gate passes.
Commits: `318275cd` holds F1–F3 together rather than one per finding; the rename from the self-check follows in a second commit.

## Self-check

- fix-consistency, LOW: the F1 test name stated only the skip outcome. Fixed: renamed to `TestCompact_HookDeliveredPointerIsSkippedAndAStaleMarkIsTyped`.
- fix-code-correctness: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
