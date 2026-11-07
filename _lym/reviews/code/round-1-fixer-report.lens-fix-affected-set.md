```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

- **F1 (probe set):** `SetInFlightProbe` is called in one place, the probe table at `housekeeping_integration_test.go:174`. The table now has the empty-answer row with `wantPack: true`, and the gc-prune subtest no longer sets a probe, so it runs on the nil probe. Both the nil probe and the empty answer are covered.
- **F2 (precondition fields):** `assertArrangePreconditions` reads three fields, `LauncherBlocker`, `CheckoutBranch` and `AdvancedToSHA`. In `livestate_verbs_test.go`, `Add` (398), `Checkout` (691) and `Pull` (906) each set their field, and the new test covers exactly those three verbs.
- **Hostile-input rows:** the `Add/<input>` and `Checkout/<input>` rows are separate cases and are not covered by the new test. `Checkout/<input>` sets `CheckoutBranch` to arbitrary input, so asserting a non-empty value there would be wrong. This is not a defect.

Tool calls:
1. One `grep` over `internal` test files for `SetInFlightProbe` and `wantPack`, to rerun F1's set of probe sites.
2. One `grep` in `livestate_verbs_test.go` for the fixture-field assignments and verb names, to rerun F2's set of verbs. My context did not show the verbs file.
