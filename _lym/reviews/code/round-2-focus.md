---
round: 2
focus:
  - "internal/fabricengine/housekeeping_integration_test.go, TestStoreHousekeeping: does some row remove a pair with no in-flight probe installed (the default nil probe) and assert the stores are packed, and does a probe-table row cover an empty answer with a nil error that also packs?"
  - "internal/fabricengine/livestate_matrix_test.go assertArrangePreconditions and livestate_verbs_selftest_test.go: does a check over the Verbs table fail when the Arrange of Add, Checkout or Pull leaves LauncherBlocker, CheckoutBranch or AdvancedToSHA empty?"
---

Both entries re-check the fold's claim that every folded assertion still stands.
