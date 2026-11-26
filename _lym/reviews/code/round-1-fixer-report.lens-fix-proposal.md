```yaml
lens: fix-proposal
outcome: reported
```

Task: fix-proposal lens, challenging the reviewer's proposals that the fixer carried out.

**BLOCKING: F5's change, `unmappedTierNames(tests, mappedTierConfigs(s.cfg))` in `internal/gatecli/reach.go#caveats`**

`lym gate reach <file>#<Func>` now tells the user that tier0 and tier1 are in no map.

- **Evidence:** `internal/gatecli/reachverb.go` `runReach` builds `step := reachStep{location: location, root: location.WorktreePath()}` with no `cfg`, although `cfg` is loaded a few lines above.
- **Path:** `runReachFunction` calls `step.describe`, which calls `s.caveats`. That runs `mappedTierConfigs` on a zero `gateslot.Config`, which returns an empty map.
- **Concrete case:** an unreached function in a package with ordinary `tiergate.Require(t, 0)` and `(t, 1)` tests prints "tests of p in tier0 and tier1, which no map holds, may run it unseen". The maps do hold those tiers. Before the fix the verb compared against the package-level `mappedTiers`, so this was right.
- **Why the gate stayed green:** it passed, so no test asserts the verb's tier caveat.
- **Fix:** pass `cfg: cfg` in that literal, and add a verb-level assertion on the caveat text (`reachverb_integration_test.go`).

I found no breakage in the proposals for F1, F3, F4, F6, F7 or F8.

Tool calls:
- `sed -n 90,200p internal/gatecli/reachverb.go`: my context held only grep lines from this file (the `reachStep{...}` literal and `runReachFunction`'s signature), not how the step is built or where `cfg` comes from.
