```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the code the fix changed (fix-code-correctness lens).

**LOW**: the manual-tier refusal in `markedScope` (`internal/gatecli/only.go`) ignores `covers`.

- Concrete input: `tier1` is `manual: true`, `tier4` has `covers: [1]`, `gate.gates.upkeep` is `{tiers: [4], scope: marked}`, and a tier1 file holds a `//lym:only upkeep` test.
- The loop tests only `slices.Contains(cfg.Gates[request.gate].Tiers, number)`, so the call is refused, although the key lists a tier whose command "runs anyway" the manual tier (`covers`).
- Before the range under review, the review and landing summary describe the rule as "is or covers", so this hub ran the test and is now refused.
- `dropCovered` and `ranWhole` in `tiers.go` both treat a covering tier as standing for the covered one.
- The behaviour matches the changed texts ("only when the key lists that tier") and the review's proposed wording ("do not list"), and the refusal's way forward (list tier1) works. The cost is therefore a narrow, recoverable over-refusal.

No other findings:

- **Undeclared marked tier:** `cfg.Tiers[...]` gives a zero `Tier` with `Manual` false, so it reaches the undeclared-tier refusal in `planTierRuns` as before.
- **Listed manual tier, or non-manual tiers:** the loop continues and the scope is unchanged.
- **Caller:** the only caller of `markedScope` is `computeScope`, gated by `isMarkedCall`, which excludes explicit targets. The signature is unchanged.
- **Manual refusal test:** it fails without the new loop, because the call would succeed. The `{tiers: [0, 1]}` step is a positive case for the way forward.
- **tier1 record assertion:** `Slots == 0` against tier0's recorded 1 tells the two tiers' settings apart. The package run with this change passed.
- **Reworded refusal in `tiers.go`:** the format arguments still match its two `%s`.

Tool calls:

1. `grep -rn -B6 'markedScope(\|isMarkedCall(' internal`: my context did not show which functions call `markedScope`.
