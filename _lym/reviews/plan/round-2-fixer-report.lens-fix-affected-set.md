```yaml
lens: fix-affected-set
outcome: reported
```

Lens: fix-affected-set. I checked that the F12 fix covers every TestGateMutate row where a short `minuteUnit` meets a real build.

Findings I left unchecked: none. F12 is the only finding.

F12 premise set: the rows that drive `mutateCmd` with a short `minuteUnit` and still run a coverage build bounded by `redundancy_build_timeout_min`.
- Only the two budget rows set a short `minuteUnit`. Every other row in the card uses the default unit, as the existing redundant tests do with `minuteUnit: time.Minute`.
- The `reach_budget_min` row starts no build. Card 5's refresh honours the spent deadline: `mapPackages` starts no package past it (`_lym/plan/05-*.md:76-77`). Its stale package is listed not judged, so no mutation build runs for it. The fix leaves it unchanged, and that is correct.
- The `mutation_budget_min` row is the one member. The fix covers it.
- Config validation allows the proposed value. `redundancy_build_timeout_min` only has to be at least 1 and has no ceiling (`internal/gateslot/config.go:288-301`), so 100000 loads.

no findings

Tool calls:
- grep of `internal/gateslot/*.go` and card 5's budget lines: my context lacked how card 5's refresh treats a spent deadline and where the build bound is validated.
- `sed` of `internal/gateslot/config.go:280-305`: my context lacked whether the build-timeout key has an upper limit that would refuse 100000.
