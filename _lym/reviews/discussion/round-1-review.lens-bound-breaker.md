```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens, attacking every bound, guard and claim about existing code in the decision record.

**Claims attacked**

1. D3: an edit errs towards no finding, and identical blocks cost at most one wrong hint. Held.
2. D2: one module-instrumented run gives both views. Held: `-coverpkg` lists the package itself (`cmd/testtiming/redundancy.go:378-382`, `:440`).
3. D4: a whole-record replace means a reader sees the old map or the new one. Held, assuming `internal/state` writes atomically.
4. D5: a package is skipped when it differs from HEAD, so the map records HEAD. **Broken**: a landing can arrive during the refresh (finding 2).
5. D5: one slot for at most `--max` packages per call. **Broken**: nothing bounds how long a test runs (finding 3).
6. D5: the per-package lock skips a package another call holds. Held.
7. D6: a package skipped as dirty waits for the next prime HEAD change. **Broken** by D6's own trigger (finding 1).
8. D6: never a second refresh while one is in flight. **Broken** across a dobby restart (finding 7).
9. D6: only the prime owner machine gets maps. Held, and stated.
10. D7 step 1: the gate's existing diff route gives the merge base and the base text. **Broken**: the route returns paths only (finding 4).
11. D7 step 1: the base the `card` key uses always exists. **Broken** for a worktree with no parent branch (finding 5).
12. D7 bound: a false flag happens only when a map test lost coverage since the map was built, and the age limit bounds that. **Broken** (finding 6).
13. D7: the step never changes the gate's result, on every failure path. Partly broken: paths are left unlisted (finding 5).
14. D7 rationale: the budget caps the step's cost. Partly broken (finding 8).
15. D7: a `//testtiming:keep` test is dropped. The escape has no stated bound (finding 9).
16. D7: the cap order is fixed and every test past the cap is listed. Held.
17. D7: a judged test is never its own cover, and a changed map test is not counted. Held.
18. D8: a content match keeps the report valid across run-record commits. Held. Across a map rebuild between the gate and the review, it is stale but still reads as valid (finding 10).
19. Constraint: no test launches a test binary. Held as stated.

**Findings**

1. **MEDIUM, design, D5 and D6** (decision-record.md:90, 95, 103, 111). Dirty-skipped packages make the prime dobby start a refresh on every tick.
   - Counterexample: an untracked `.go` file, or one dirty tracked file, sits in a package directory in the prime worktree.
   - D5 counts that package stale (line 85), skips it (line 90) and reports it unbuilt (line 95).
   - D6 starts a refresh whenever "the last refresh reported stale packages left unbuilt" (line 103).
   - Each tick therefore starts another `lym gate testmap`. Each one takes a gate slot, builds `lym` when a `tier1` test runs, and skips the package again.
   - This contradicts D6's bound that a dirty-skipped package "waits for the next prime HEAD change" (line 111).
   - Fix: leave dirty-skipped packages out of the unbuilt count that retriggers a refresh, or retrigger only on unbuilt packages that were not skipped.

2. **MEDIUM, design, D5** (decision-record.md:90; `internal/fabricengine/merge.go:418`, where landing runs `merge --ff-only` in the parent's weft checkout).
   - A landing into the prime during a refresh changes the package files under a running map build.
   - The dirty check happens once, before the run. The map records HEAD as its commit and hashes the files, but tests may have run over the other tree.
   - The result is a map with the wrong coverage that reads as fresh until the next HEAD change or the age limit.
   - D5 states no ordering between a refresh and the parent lock or the weft write lock, and it does not re-check HEAD and hashes after the run.

3. **MEDIUM, design, D5** (decision-record.md:92, 97).
   - `testmap` sets no per-test timeout; D7 has one at line 124, D5 has none.
   - A hung `tier1` test in the prime holds the slot indefinitely, and D6's "no refresh in flight" (line 103) then blocks every later refresh.
   - The bound "holds one slot for at most `--max` packages" bounds package count, not time.

4. **MEDIUM, design, D7 step 1** (decision-record.md:120; `internal/gatecli/scope.go:476-494`, `cardChanges` → `impactset.ChangedPaths`).
   - The gate's existing diff route returns changed paths only.
   - Step 1 needs each test function's text at the merge base, which needs a file read at a revision. The record names none; `fabricengine.AnchoredFileAt` exists but is anchored to records.
   - Under `PATTERN-fabric-boundary`, git on weft goes only through `fabricengine`.
   - The claim that the existing route serves step 1 is false. The record leaves the means to Plan.

5. **MEDIUM, design, D7 failure paths** (decision-record.md:120, 131-132; `internal/gatecli/scope.go:668-678`).
   - `parentMergeBase` reports false for a worktree with no parent branch, such as the prime on main. The step's behaviour in that case is undefined.
   - The listed failures also omit an unreadable or corrupt map record, a failed report write and an unreadable merge base.
   - "Never changes the gate's result" is asserted only over the listed paths, and no acceptance criterion reaches the unlisted ones.

6. **MEDIUM, design, D7 bound** (decision-record.md:125, 138).
   - The claim that a false flag happens "only when a map test lost coverage since the map was built, which the age limit … bounds" is wrong.
   - Counterexample: the diff edits a non-test helper or a branch condition, so map test `TestA` no longer reaches block B. B's text is unchanged, so its key is unchanged, and `TestA`'s function hash is unchanged, so `TestA` still counts as a cover.
   - A new test covering B is then reported contained, although after the diff nothing else covers B.
   - The same holds when a `_test.go` helper outside `TestA`'s own function changes.
   - The age limit does not bound this: the drift comes from the diff itself, not from time.

7. **LOW, design, D6** (decision-record.md:103, 106).
   - The dobby keeps its in-flight state in memory. A dobby restart while a refresh runs loses it, and the new dobby's first tick starts a second refresh (line 105).
   - The per-package lock keeps the two from building the same package, but two slots are held.
   - The orphaned child has no stated owner or end path; `PATTERN-child-processes` puts a hub process under `runproc.HubOwner`.

8. **LOW, design, D7 cost** (decision-record.md:124, 136).
   - The budget stops starting tests, but a test already started can run up to `redundancy_test_timeout_sec` past it.
   - Per-package coverage builds and the `lym` build are not counted against the budget, and up to `redundancy_max_tests` packages can each need one build.
   - "Capped by the budget" understates the step's worst case.

9. **LOW, design, D7 step 2** (decision-record.md:121).
   - A `//testtiming:keep` line on a new test removes it from judgment.
   - D7 does not state what bounds that escape, for example that the keep line and its reason show in the reviewed diff, or that the report lists such tests as not judged so review sees them.
   - Step 2's "listing each with its reason" covers it only implicitly.

10. **NIT, design, D8** (decision-record.md:143).
    - The match ignores the map. When a refresh rewrites the map between the gate run and Code-Burler-Review, the report still matches and shows contained verdicts computed against the old map.
    - The cost is bounded because review reads each finding, but the record does not state that bound.

Tool calls: none.
