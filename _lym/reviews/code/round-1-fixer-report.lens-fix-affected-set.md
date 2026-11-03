```yaml
lens: fix-affected-set
outcome: reported
```

Task: the fix-affected-set lens, run against F1 (isBeam fallback branches untested).

No findings.

F1's premise is a set, the two fallback branches of `isBeam` at `internal/ideengine/spawn.go:113`. The branches are an absent record (`found == false`) and a `ReadOriginFor` error. The new check `checkBeamWithoutReadableOriginOpensBareFolder` covers both:
- **Error branch:** it removes the weft `.weft` lock directory, the error path at `origin.go:134`.
- **Absent branch:** it removes the origin record, the `found == false` return at `origin.go:126`.

The search for other sites on this key, `isBeam(` callers, found a single caller, `spawn.go:96`. Both the `Spawn` and `Seed` paths reach it through `seed`, so one check covers the set. The check also asserts that no beam workspace file is written.

One thing the check does not exercise: `Seed` on those two degrade rows. `Spawn` already runs `seed`, so this is not a gap.

Unchecked: nothing.

Tool calls (1 of 6):
- `grep -rn "isBeam(\|ReadOriginFor(" internal`, to list every caller of `isBeam` and `ReadOriginFor`. My context held only `spawn.go` and the ideengine tests.
