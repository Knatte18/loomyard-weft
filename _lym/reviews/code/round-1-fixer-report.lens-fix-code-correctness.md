```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the F1 and F2 code changes.

**Findings**

- LOW. `housekeeping_integration_test.go`, the comment above the probe table: "like the nil probe the gc-prune subtest below runs on".
  - The subtest was renamed to "Remove on the default nil probe packs the stores, …".
  - No subtest is called "gc-prune" any more, so the pointer is stale. This is a naming slip only, with no effect on behavior.

**Walk-throughs**

- **Empty-answer row:**
  - Input: `probe` returns `(nil, nil)` and the threshold is 1.
  - `fillSampledLooseObjects` leaves more than 1 loose object in objects/17, and `Remove` finds no other-pair sessions, so it runs the gc.
  - `sampledLooseObjects` becomes 0, so `packed` is true and matches `wantPack`.
  - If `Remove` skipped the gc on an empty answer, `packed` would be false and the row would fail.
  - The row's empty `wantLog` skips the log check correctly.
- **Nil-probe subtest:**
  - After the `SetInFlightProbe` line was dropped, the probe is whatever `hubforge.NewHub` installs, which the review says is nil. The reviewer's description of the old row as running on the nil probe supports this.
  - If the default stopped packing, the `n != 0` assertion would fail, so the pin is restored.
  - Both the packed assertion and the prune/keep assertions are unchanged.
- **`TestVerbCases_ArrangeSetsItsPreconditionField`:**
  - Matching on exact `Name` means the hostile-input cases ("Add/…", "Checkout/…") are not picked up.
  - `vc` points into `Verbs`, so it is stable across the parallel subtests, and each subtest builds its own hub.
  - If an Arrange left its field empty, the test would report that. All three Arranges set the field at `livestate_verbs_test.go:398, 691, 905`, so it passes today.
  - If `Arrange` fails, it fails through `t`, as expected.
  - The `Verbs` case Names read for Add, Checkout and Pull are `"Add"`, `"Checkout"` and `"Pull"` (lines 389, 676, 892).
- **Callers:** no signature changed. `assertArrangePreconditions` is untouched.

**Tool calls:** none. My context already held every changed file.
