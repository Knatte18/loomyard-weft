```yaml
lens: fix-code-correctness
outcome: reported
```

No findings for lens fix-code-correctness.

Traced inputs and callers:
- **`ledRole`:** `""` gives `burler-review`, and `"code"` gives `code-burler-review`. `Profile.rolePrefix` still serves `fix.go:75` unchanged.
- **Chair role:** `reviewTable` now builds its prefix through `ReviewChairRole(p.RolePrefix)`, which equals the old `p.rolePrefix(ReviewRolePrefix)`. The chair's real strand role (`SeatRole(prefix, "")`) therefore matches the role `segmentAssistBounds` looks up.
- **`baseRole` lookup:** `baseRole("code-burler-review")` returns the role unchanged, because `review` is not a numbered suffix. `keep_warm_cap_roles` is therefore keyed by the full `code-burler-review`, as the test fixture assumes.
- **Fallback:** a `Format` failure leaves `shuttleengine.DefaultAwaitCap`, as before.
- **Caller:** `loomSegmentSettings` has one caller, the `wire` call site, which was updated. `assistBounds[perch]` is filled for every perch in `loomSegmentPerches`, so no lookup falls through to a zero value.
- **Fixture (`wiring_test.go`):** with the old single bare-role cap, the 2m `burler-review` entry would give every segment `10m − allowance − 2m`. That fails the Discussion and Plan expectations (250s cap) and the Code expectation (3m). The tier0 gate passed with the new block, so the room stays positive (`10m − StartAllowance − 250s > 0`).
- **F2 (`zeroMeans`):** `"must not be negative"` is kept, so the assertions at `config_test.go:613` and `:655-657` still hold.

Calls made:
1. `Read internal/shuttleengines/keepwarm.go`: to confirm how `KeepWarm` derives the role (`baseRole`) and when it falls back. My context did not hold it.
2. `Bash grep StartAllowance` and `CapAssistTimeout`: to check the test's arithmetic and that a non-positive room cannot pass silently. My context did not hold either.
