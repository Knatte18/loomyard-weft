```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1/F2 change set for consistency with its surroundings (lens fix-consistency).

No findings.

- F1: `hubforge` never installs a probe, so `NewHub` leaves `t.inFlight` nil and the gc-prune subtest runs on the nil probe again, as before the fold.
- F1: the probe-table comment now says an empty answer packs "like the nil probe the gc-prune subtest below runs on". The new table row and the renamed subtest agree with it.
- F1: the `wantPack` field and the `wantLog` guard already handle the new row, which has `wantPack: true` and an empty `wantLog`.
- F2: the new test reads `LauncherBlocker`, `CheckoutBranch` and `AdvancedToSHA`, the same three fields `assertArrangePreconditions` guards with `!= ""`.
- F2: the verb names "Add", "Checkout" and "Pull" match the `Name:` values in the `Verbs` table. Tier 0 and tier 1 gates were green.

Calls made:
1. `grep` for `SetInFlightProbe` and `inFlightProbe` across `internal` — to see whether anything installs a default probe that would break the nil-probe premise of the gc-prune subtest.
2. `grep` for "probe" in `topology.go` and for `SetInFlightProbe` in `internal/hubforge` — to confirm the nil default and that `hubforge` does not install a probe.
