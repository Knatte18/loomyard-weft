```yaml
lens: fix-code-correctness
outcome: reported
```

Task: correctness check of the code the fix changed.

**BLOCKING**: `internal/gatecli/reach.go#caveats`, the new call `unmappedTierNames(tests, mappedTierConfigs(s.cfg))` (F5).

`lym gate reach <file>#<Func>` now reports every tiered test of the package as being in a tier no map holds.

- **Cause:** `runReach` builds `step := reachStep{location: location, root: location.WorktreePath()}` with a zero `cfg` (`internal/gatecli/reachverb.go`). It passes that step to `runReachFunction`, which calls `step.describe` and so `caveats`. `cfg` is loaded a few lines earlier but never put on the step.
- **Effect:** `mappedTierConfigs(zero Config)` is empty, because `cfg.Tier(n)` errors for every n.
- **Concrete case:** take an unreached function in a package with ordinary tier0 and tier1 tests, in a hub declaring both. The verb prints "tests of p in tier0 and tier1, which no map holds, may run it unseen", which is false. Before the fix it used the `mappedTiers` constant and printed no such line.
- **Why the gate stayed green:** `TestUnmappedTierNames` passes the map directly, and the step's integration test uses a step that has a `cfg`. No test asserts the verb's caveat lines.
- **Fix direction:** set `cfg: cfg` on the verb's `reachStep` literal, and add an assertion on the verb's caveat lines, for example in `reachverb_integration_test.go`.

No other findings. I traced these and they hold:

- the older-format branch of `staleLinked`
- the linker-test loop in `caveats`, which copies each struct and so leaves the reader's slices unchanged
- `unseenCause`
- `UnseenLinkerTests`
- the new rows for F1, F2, F3, F4 and F6, each of which fails with its defect in place; the five-test row only pins the cap boundary

Tool calls:

1. I read `internal/gatecli/reachverb.go` (lines 90-200). My context showed only grep lines for the verb's `reachStep` literal and its `describe` path.
