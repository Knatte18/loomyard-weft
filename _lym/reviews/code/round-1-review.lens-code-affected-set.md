```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, meaning what the diff forces outside the lines it touches.

**Per change, the set I found:**

1. **`NestedPackageDirs` moved from `gatecli` to `testcover`, exported.**
   - Set: `internal/gatecli/testmaptree.go:32` is the caller, now switched to `testcover.NestedPackageDirs`. `internal/testcover/testselect.go:317` is the new caller.
   - A repo-wide grep for the old and new names finds no other reference.
   - `buildableDir` has no use left in `gatecli` and exists only in `packagedirs.go`.
   - The `maps` import was dropped from `testmaptree.go`.
   - Covered. No test named the old function.

2. **New `testcover` surface: `Selector`, `NewSelector`, `SelectorConfig`, `CommitReader`, `Miss`, `MissReason`, `Selection`.**
   - Set: `internal/testcover/doc.go` is updated with a Selector paragraph and miss list. `docs/overview.md` has the `testcover` row updated.
   - The other markdown that names `internal/testcover` is `pattern/PATTERN-test-speed.md` and the `docs/research/...` reports.
   - `PATTERN-test-speed.md` describes the map's contents and rebuild rules, and the diff changes none of those.
   - The research reports are dated records and are not rule documents.
   - No PATTERN entry is declared changed in my directive context, so "Declared PATTERN changes landed" has nothing to check.
   - Covered.

3. **`CodeFileAt` read through the one-method `CommitReader` interface; `*fabricengine.Fabric` satisfies it.**
   - Set: `fabricengine/codereads.go:40` has a matching signature.
   - `testcover` imports no new fabric package, and no new git call is made.
   - This fits PATTERN-fabric-boundary and the record's constraint.
   - Covered.

4. **Repository rules checked (PATTERN-code-conventions, PATTERN-test-speed, PATTERN-tolerant-records).**
   - `doc.go` documents only this package.
   - The two directive strings `//go:embed` and `//go:` are each one named constant.
   - `ReadMap` is reused, so no new reader of the map file and no unknown key is refused.
   - The new test opens with `tiergate.Require(t, 0)`, is parallel, uses a fake commit reader, spawns nothing and uses `t.TempDir`.
   - Covered.

5. **Commit messages and departures.**
   - Neither commit carries a `Pattern-Departure:` trailer, and I found no difference from the record that one would need to excuse.
   - The unrecorded design choice in `_lym/landing/summary.md` is that spawn-marked tests are selected whenever any linked function changed. That is a handler scope matter, outside this lens.
   - Covered.

**Findings:** none under this lens.

**Tool calls (1):**
- `grep -rln "internal/testcover"` over `.md` and `.yaml` files, to list docs and rule files that describe `testcover`. My context lacked that list.
