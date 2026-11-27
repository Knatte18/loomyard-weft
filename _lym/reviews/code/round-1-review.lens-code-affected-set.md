```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens. For each change the diff makes, I looked for everything outside the touched lines that calls, reads or asserts it.

**List: per change, the set found, covered or not**

1. **`TestAlphaHangs` renamed to `TestAlphaHangsOnCue` in alpha's fixtures** (`testmap_integration_test.go:860`)
   - Set: the `AnAllCall…` wants for outcome, timed-out list and map entry (1069–1071, 1081); the `NamedPackageIsRebuiltWhenFresh` want (1139); the `AMapRunBuildsNoLym` want (1336); the `AnEllipsisTarget…` exit comment (1231); the packageTests comment (857–858).
   - All covered. `git grep` finds no other reference to `TestAlphaHangs`, `HangsOnCue`, `NamedPackageIsRebuiltWhenFresh` or `AMapRunBuildsNoLym` anywhere in the repository.

2. **New stand-in suffix `HangsOnCue`** (`runTestMapStandIn`, 303–309)
   - Set: the stand-in doc comment (236), which was updated.
   - The existing `Hangs` users are unchanged, as the summary says: `TestYieldingHangs` in `TestGateTestMapCLI` (2086) and `TestDropsHangs` in `redundancy_integration_test.go:94/278/478`.
   - The `HangsOnCue` case comes before the `Hangs` case. The suffixes cannot both match one name, so the order does not matter.
   - Covered.

3. **New const `testMapSecondUnit` replaces every `150 * time.Millisecond` in this file** (663; call sites 653, 818, 923, 1287, 1414, 1580)
   - Set: the `TestGateTestMap` header comment (838) was updated.
   - No `150` literal is left in the file.
   - `secondUnit` feeds only `testTimeout` in the map run (`testmap.go:338`, then `testmappackage.go:559`). No build, mutant or call bound in these calls derives from it.
   - It is a `const`, so it does not break PATTERN-test-speed's rule against a package-level `var` (`pattern/PATTERN-test-speed.md:174-175`). `t.Parallel` is unaffected.
   - The sibling steps in `reach_integration_test.go:176` and `redundancy_integration_test.go:247` keep their own 150 ms units. They are not this test's map calls, so nothing forces a change there.
   - Covered.

4. **`callCounts` signature gains an `env` parameter** (932)
   - Set: its one caller (1064), which was updated. Covered.

5. **Outputs and file shapes**
   - The diff changes no production output, key, message or record format, and touches no stencil, spec or doc.
   - Covered: there is nothing to update.

6. **Rule documents and commits**
   - PATTERN: no declared changes, and the diff edits no PATTERN.md or `pattern/` file.
   - Commit messages: `Root-Cause` names the shared 3 s limit (20 × 150 ms), which is the defect the diff removes. Neither commit carries a `Pattern-Departure:` trailer, and no departure from the answer key needs one.
   - Covered.

**Findings:** none.

**Tool calls**
- `git grep` for `TestAlphaHangs|HangsOnCue|NamedPackageIsRebuiltWhenFresh|AMapRunBuildsNoLym` outside the test file, and for clock/timeout rules in `pattern/PATTERN-test-speed.md`. My context had no repository-wide search for references to the renamed and changed symbols, and the test-speed background had not been read.
