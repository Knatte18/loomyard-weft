```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals for F1 and F2, both fixed as proposed.

No findings.

- **F1 (nil probe row):**
  - Dropping `SetInFlightProbe` from the merged row restores the pre-fold setup, and the tier-1 gate passed with its packed (objects/17) and prune assertions intact.
  - The new "an empty answer packs the stores" row sets `wantPack: true` with `wantLog: ""`. The shared body only checks the log when `wantLog` is non-empty, so the row asserts keys and packing and nothing else.
  - The probe is set after `AddPair`, as in the other rows, so `Add` is unaffected.
  - I found no case where the split is wrong. The nil probe and the empty answer are each pinned by a row that asserts packing.
- **F2 (Arrange self-check):**
  - The new test fails if an Arrange stops setting its field, because it asserts non-empty.
  - `Verbs` names are unique for `Add`, `Checkout` and `Pull`; the hostile-input cases use `Add/…` and `Checkout/…`, so the exact-name lookup cannot hit the wrong case.
  - Each subtest builds its own `CopyHub`, so the parallel subtests share no state.
  - The test takes `&Verbs[i]` and assigns without breaking, so the last match would win. That is harmless because names are unique.

Further tool calls: none.
