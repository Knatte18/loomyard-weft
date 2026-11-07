# Fold the overlapping internal/fabricengine tests the tier 1 report read

This folds the overlapping `internal/fabricengine` tests named in section 4 of the tier 1 revision report (`docs/research/speed/tier1-test-revision-2026-11-06/report.md`).
Each fold runs fewer hubs or fixtures, and none drops an assertion.
Each fold is its own commit, and that commit's message lists every assertion the folded row alone made, matching the report's "Alone pins" column.
Only test files change; production code is untouched.

## The folds

- **`TestVerbCases_CleanState` is gone.**
  `TestCrossProduct` already runs every verb in the clean state at both anchors through the same build, arrange, state, capture, run and no-unpermitted-change sequence.
  The test's three preconditions move into `runCell` as `assertArrangePreconditions`, checked right after the arrange and before the state is applied:
  Add's launcher blocker is a directory, Checkout's branch differs from the one checked out, and Pull's arrange advanced the warp bare.
  `livestate_verbs_selftest_test.go` keeps only `TestVerbCases_StatesRestrictionIsWellFormed`.
- **`TestAnchorChange`**: the three refusal rows that plant nothing (a target the validator rejects, a reserved segment, an untracked directory) run in turn on one `hubforge.CopyHub`.
  The refusal assertions move into `requireAnchorRefusal`, which the planted rows use too.
- **`TestStoreHousekeeping`**: "Remove packs a store above the lowered threshold" and the probe's "an empty answer packs" row are deleted.
  The gc-prune row installs the empty-answer probe and now also asserts that the stores are packed and that both housekeeping keys are written.
- **`TestReconcileOrigin_Moves`**: the `owned_elsewhere` subtest's six replies run on one `movesPair`.
- **`TestPushFailureClasses`**: `ThroughPublishesPath` is deleted.
  Both settle rows assert its hold-back on their first push, through `requireRewoundHoldBack`.
  The two pushes to an unreachable origin from a branch with an upstream become one two-row table, one row for `CoalescePushRequest` and one for `PushAnchored`.
- **`TestBolt_PullThenCommitWritten`**: three subtest pairs become two-row tables:
  the behind-only pulls, the seed drops, and the diverged conflicts.

## Design decisions

- `assertArrangePreconditions` keys each check on a fixture field that only one verb's arrange sets, such as `LauncherBlocker` or `AdvancedToSHA`, rather than on the verb's name.
  This keeps the cross-product loop free of per-verb branches, as its file header requires.
- The shared-hub refusal rows in `TestAnchorChange` run one after another, not in parallel.
  They share one hub, and each row checks that the weft's HEAD, status, wiring and move record are unchanged.
- In `TestStoreHousekeeping`, the gc-prune row uses the empty-answer probe instead of no probe.
  That way it covers the probe-consulted packing path.
  The nil-probe default still runs under the subtests that check the key writes and the failing gc.
- `TestPushFailureClasses`' "backoff rows" are read as the two transport-record rows that wait out the whole backoff.
  They differ only in the entry point and the error it returns, which is what the report says they alone pin.
  The `PushAnchored` row now also asserts one mail, as every failed-push record write sends.
- In `TestBolt_PullThenCommitWritten`, a few assertions that held for one row of a pair now run on both, where they hold on both:
  - the upstream-ancestor and head checks on the two-write row;
  - the clean-board, empty-record and logged-once checks on both diverged rows.
- The diverged way-forward text stays on the row without a seed commit, since the detail names every commit ahead.
