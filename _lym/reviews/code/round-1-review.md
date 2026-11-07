---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/housekeeping_integration_test.go:190
    summary: The fold loses the pin that Remove with no in-flight probe set packs the stores, which the decision says no fold may lose.
    fix: In the gc-prune subtest of TestStoreHousekeeping, drop the h.Topology.SetInFlightProbe line so the row runs on the default nil probe as it did before the fold, and keep its key-write, packed (objects/17) and prune assertions. Re-home the empty-probe packing as a wantPack true row in the probe table (an empty answer, a nil error) so both the nil probe and the empty answer still pack. Name the nil-probe assertion in the commit message.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/livestate_matrix_test.go:195
    summary: assertArrangePreconditions skips each check when its fixture field is empty, so an Arrange that stops setting the field leaves a column vacuous and nothing fails.
    fix: Add a check in livestate_verbs_selftest_test.go, over the Verbs table, that the Arrange of Add, Checkout and Pull returns a non-empty LauncherBlocker, CheckoutBranch and AdvancedToSHA respectively. Or drop the != "" guards in assertArrangePreconditions and give VerbCase a flag naming which preconditions its Arrange must meet.
---

### [MEDIUM] Nil-probe packing no longer asserted after the housekeeping fold

**Location:** `internal/fabricengine/housekeeping_integration_test.go:190`
**Issue:** The deleted "Remove packs a store above the lowered threshold" row was the only row that removed a pair with no probe installed and asserted the stores were packed. The old gc-prune row also ran on the nil probe. The merged row calls `SetInFlightProbe` with an empty answer, so every remaining packing assertion goes through an installed probe. The report's alone-pins column names "that the default probe packs", `housekeepStores` documents "A nil probe reports none", and the decision requires every folded row's assertion to stand. The change description claims the nil-probe default still runs under the key-write subtests, but those lower no threshold and assert no packing. Commit 303684b1 does not name the dropped assertion.

### [LOW] Precondition checks skip when the fixture field is empty

**Location:** `internal/fabricengine/livestate_matrix_test.go:195`
**Issue:** The deleted test read `LauncherBlocker`, `CheckoutBranch` and the Pull SHAs unconditionally, so an Arrange that stopped setting one failed it. The new checks run only when the field is non-empty. Today every Arrange sets its field (`livestate_verbs_test.go:398, 691, 905`) and the Checkout and Pull Effects back two of the three for the clean state, so the gap is latent. It is the vacuity the preconditions exist to catch, and the commit's claim of keeping every assertion is slightly too strong.

## Rejected

- Checkout, Pull and Add checks now run in every state's cell, not only clean: stricter than before, not a defect.
- Shared `logcapture.Capture` across the two parallel diverged rows: the log-count check runs on one row only and the detail names that board's own tip, so it is safe.
- `owned_elsewhere` six replies on one pair: the assertions per reply stay, `asked` is reset per iteration, and no origin moves.
- `TestAnchorChange` unconditional `tt.setup`: every remaining planted row sets it.
- Shared-hub refusal rows run in turn on one hub: each row re-reads its HEAD, status and record baselines, and the refusals plant nothing.
- Each test keeps one `tiergate.Require` at tier 1; the folds add none and remove only the deleted test's.
