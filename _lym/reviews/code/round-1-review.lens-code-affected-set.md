```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens. Per change, the set of callers, readers, asserters, docs and rules I found:

- **`TestVerbCases_CleanState` removal and `assertArrangePreconditions`:**
  - Remaining references to the deleted test and to `cleanStateEntry` are all measurement or research documents (`docs/benchmarks/*`, `docs/research/speed/*`), which record past runs and are exempt.
  - `livestate_verbs_selftest_test.go` has its header rewritten and its unused imports removed. Its one remaining test and `tiergate` import are intact.
  - The "clean-state" comments in `livestate_verbs_test.go` (lines 169, 384, 604, 651, 807) describe Effect closures, not the deleted test, so they are not stale.
  - The `Verbs` fixtures that set `LauncherBlocker`, `CheckoutBranch`, `OriginalBranch`, `AdvancedFromSHA` and `AdvancedToSHA` (lines 394-398, 630, 690, 905) are covered by the new check. Pull's Effect closure at line 947 and Checkout's at line 763 still assert the same conditions, so the new check is redundant for them.
  - The `!= ""` guards in `assertArrangePreconditions` are looser than the old unconditional checks. The old code failed if Pull's Arrange left `AdvancedToSHA` empty, since from equalled to. The new code skips that case silently. An Arrange that stopped setting `LauncherBlocker`, `CheckoutBranch` or `AdvancedToSHA` would no longer fail. Add and Pull are covered, only by the other check in each (Pull's Effect closure at line 947). Not covered: nothing requires that Add's, Checkout's and Pull's Arranges set their fields at all. This weakens "keeps every assertion its row alone made".
  - Pull is now also checked in cells where the state may have omitted or disturbed it, which is fine because the check runs before the state is applied.
- **`TestAnchorChange`:**
  - `requireAnchorRefusal` is used by both the shared-hub rows and the planted-setup table.
  - The planted-setup table now calls `tt.setup` unconditionally. The remaining rows I saw (the empty anchor file and `ln` rows) set `setup`. I did not re-read the rest of the table, so I cannot confirm that no remaining row has a nil `setup`.
  - Covered: the shared-hub rows share the helper. Not verified: the nil-`setup` risk.
- **`TestStoreHousekeeping`:**
  - The deleted "Remove packs a store above the lowered threshold" row was the only one that packed with the default nil probe. The "an empty answer packs" row used an explicit empty probe, and the gc-prune row used the default one.
  - The fold's commit and `summary.md` say the nil-probe default still runs under the key-write subtests, but those subtests neither lower the gc threshold nor fill loose objects. They cannot assert packing.
  - After the fold, packing with no probe set is asserted nowhere. The report's alone-pins column ("that the default probe packs") is lost. Not covered.
- **`TestReconcileOrigin_Moves`:** the six replies now share one `newMovesPair`. The commit message names `OwnerOf` asked once, about `task`. `asked` is reset per iteration. Covered.
- **`TestPushFailureClasses`:**
  - `requireRewoundHoldBack` replaces the deleted `ThroughPublishesPath`, and both settle rows call it on their first push.
  - The backoff table adds a mail assertion to the `PushAnchored` row, as `summary.md` says.
  - Both settle rows previously used `_` for `originHead` and `dropped`; the diff now passes them through. Covered.
- **`TestBolt_PullThenCommitWritten`:**
  - The merged diverged table's seed row now also asserts the clean board and empty record. The non-seed row now also asserts the same two things, which the old row did not. That is added coverage, not a loss.
  - The logger capture is shared across the parallel rows. Covered.
  - The "behind only" table is covered.
  - The old "two writes" row never asserted the upstream ancestor or head; the new one does. That is added coverage.
- **PATTERN rules:** `PATTERN-test-speed` and `PATTERN-test-isolation` need no PATTERN or `pattern/` change from this diff. The decision record declares no PATTERN changes for D6. No commit carries a `Pattern-Departure:` trailer, and none is needed.

### Findings

1. **MEDIUM, design:** The default-probe packing assertion is lost.
   - Location: `internal/fabricengine/housekeeping_integration_test.go:190-197`.
   - The fold deleted "Remove packs a store above the lowered threshold". It was the only row that removed a pair with no probe installed and asserted the stores were packed. The gc-prune row now installs an explicit empty-answer probe, so no remaining test asserts packing when no probe is set.
   - `summary.md` claims "the nil-probe default still runs under the subtests that check the key writes", but those subtests do not lower the threshold or assert packing.
   - The commit message for 303684b1 does not name this assertion, and the record requires each fold to keep every assertion its row alone made.
   - Fix: keep the gc-prune row on the default nil probe and add the empty-probe packing as a separate assertion or row, or restore a nil-probe packing assertion.
   - Evidence: the diff hunk deleting the row, and the `SetInFlightProbe` line added to the gc-prune row.

2. **LOW, design:** `assertArrangePreconditions` skips when its key field is empty.
   - Location: `internal/fabricengine/livestate_matrix_test.go:195-212`.
   - The old test checked unconditionally per verb. The new guards `LauncherBlocker != ""`, `CheckoutBranch != ""` and `AdvancedToSHA != ""` pass silently if an Arrange stops setting them, which makes a column vacuous without a failure.
   - Fix: require the field set when the verb is Add, Checkout or Pull. That needs `runCell` to know the verb, or the fixture to carry a marker. Otherwise accept the loss and record it in the commit.
   - Evidence: the function body, against the deleted `switch vc.Name` block that read the fields unconditionally.

Tool calls:
1. Grep for references to the deleted tests and helpers across `.go` and `.md` files; my context lacked repo-wide references outside the diff.
