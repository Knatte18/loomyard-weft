```yaml
lens: interactions
outcome: reported
```

Task: review the decision record's decisions in pairs, checking where they act on the same state, key, command or invariant.

Pairs checked:
- D1 and D3, guard tests against narrowing: **broken** (F1).
- D1 and D3, `tree_scope` against narrowing: **unclear** (F4).
- D1 and D5, the import scope for tier 2 against "only removes tests": held.
- D1 and D5, `inert_paths` against the non-Go-file miss: held.
- D2 and D5, the format marker as testmap staleness and as a scope miss: held.
- D2 and D5, cross-package file hashes as a scope miss but not as testmap staleness: **broken** (F2).
- D2 and D3, failed, spawning or zero-block tests left out of the map and therefore always run: held.
- D2's redundancy promise against cross-package map tests: held, since the own-package filter makes a zero-own-block map test cover nothing.
- D3 and D5, importer files at base against the map's own-package hashes: held.
- D4 and D5, removed base keys against the rule that recorded hashes match the base: held.
- D4 and D5, a changed package the importer's map covers no block of, so nothing is selected: held within D5's stated drift bound.
- D4 and D5, a renamed plain function selecting nothing while its callers' blocks do: held. A renamed method widens: held.
- D3 and D6, the selected set against the `-run` union superset: the decisions hold, but D3's acceptance criterion contradicts D6 (F3).
- D6 and D1, the green record against the per-tier `newestWholeAncestor` base: held. Whole-run and miss packages record package entries, the selected invocation records none, and the tier is never whole.
- D6 and the failed-test rerun (`FailedTestsCall`, explicit targets, no scope computation): held.
- D6 and `//lym:watch`: held. The watch skip acts inside the test whether the test is run by name or by package.
- D7 and D3, "none selected" against unrecorded tests: held.
- D8 and D5, a torn map read counting as unreadable, so a miss: held.
- Constraint PATTERN-gate-slot against D8, reading maps with no lock: held.

Findings:

1. **MEDIUM**, D1 and D3: guard tests are lost in narrowed importing packages.
   - D1 says `//lym:guard` tests act as under `diff`. But the guard scan only lists guard tests in directories **outside** the scope (`scope.go`: "each directory outside the scope that holds its guard tests").
   - A guard test inside an importing package depends on that package running whole. Under D3 that package runs only its selected tests, so the guard test runs only when the map happens to select it.
   - D3's union should include every guard test of a narrowed package, and an acceptance criterion should pin it.

2. **MEDIUM**, D2 and D5: testmap and the scope disagree on when a map is fresh.
   - D5 makes a map a miss when any cross-package file hash it records differs from the base.
   - D2 changes testmap's staleness only by adding the format marker. testmap still judges a map stale by its own package's files at HEAD and by age.
   - So drift in a covered dependency is a permanent scope miss that testmap never repairs until the 7-day age rebuild. That gap is part of D2's own decision, not only `testmap-upkeep`'s policy.
   - D2 should say whether testmap's stale rule also checks the recorded cross-package hashes.

3. **LOW**, Acceptance criteria D3 against D6: the D3 criterion says an importing package "runs exactly the selected tests plus its tests the map does not record". D6 allows a superset when a name selected in one package matches a same-named test in another. Write "at least" or "the selected set, plus D6's same-name overlap".

4. **LOW**, D1 against Darn's `tree_scope`: D1 says `tree_scope` acts as under `diff`, but it does not say whether `tree_scope`'s explicit targets are then narrowed by D3 to D5 or run whole. They come from `darn.yaml` rather than through `computeScope`, so the reading is ambiguous.

Tool calls:
- Grepped `guard` in `internal/gatecli/scope.go`: my context did not show whether guard tests are scanned inside or outside the scope.
