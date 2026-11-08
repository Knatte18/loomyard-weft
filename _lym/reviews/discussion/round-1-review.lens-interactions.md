```yaml
lens: interactions
outcome: reported
```

Lens `interactions`: check every pair of decisions that act on the same state, then check the acceptance criteria and constraints against each decision they touch.

**Pairs checked:**
- D1 × D2: the engine verdict is shared, and the tool passes module-wide `-coverpkg` and filters to the package view. Held.
- D1 × D5/D7: the engine binary's `-coverpkg` differs per caller (package-only for the map and the step). D1's eligibility rules are unchanged. Held.
- D2 × D4: the tool runs module-wide while the map runs package-only. Both give the package's own blocks, because cover instruments per file. Held.
- D3 × D4: the map stores the D3 key table. Held.
- D3 × D7: the step keys fresh profiles from current files and the map from its commit's files. The ordinal mismatch stays within D3's stated bound. Held.
- D4 × D5, staleness by file hash against the dirty skip against HEAD: an untracked Go file is always "differs from HEAD". Broken (finding 2).
- D5 × D6, the "stale packages left unbuilt" trigger against D5's dirty and lock skips: Broken (finding 1).
- D5 × D6 when a dobby restarts with a refresh still running: the in-flight state is lost, the second call's lock skips feed finding 1, and the double run is otherwise bounded by the lock. Held apart from finding 1.
- D5 × D7, the map's whole replace while the step reads it: a reader sees the old map or the new one. Held.
- D5 × D7, tier `env` and the `lym` build: both use `gate.yaml` tier `env`. Held.
- D6 × D7, maps only on the prime dobby's machine: D7 reports "no map" elsewhere, as stated. Held.
- D7 step 2 × D7 step 4 × D1, when eligibility is known: skipped, failed and no-block tests are only known after the run, but step 2 drops them before it. Broken (finding 4).
- D7 × D8, the red-gate report against D8's match test: Broken (finding 3).
- D7 × D8, Parent-Align moving the merge base: step 1's set changes, so D8 fills "no current report". That is by design. Held.
- D7 step 5 × D4, function hashes and map tests that still exist: Held.
- D7 × D9 and the declared `PATTERN-test-speed` text (the report checks, advisory): Held.
- Constraint "no test launches a test binary" × the D1 to D7 criteria: Held.
- Constraint `PATTERN-gate-slot` × D5's one slot for the whole call: Held.
- The D5 acceptance criterion "rebuilds ... a changed or untracked file" against D5's dirty skip: Broken (finding 2).
- The D6 acceptance criterion "starts a refresh ... when stale packages were left" against D6's dirty-skip bound: Broken (finding 1).
- All other criteria (D1, D2, D3, D4, D7, D8) against their decisions: Held.

**Findings:**

1. **MEDIUM, design. D5 × D6, `decision-record.md:90-95` and `:103`, `:111`.** The refresh restarts on every dobby tick while any stale package is skipped.
   - D5 skips a package that is dirty against HEAD (`:90`) or whose lock another call holds (`:91`), and it "exits reporting how many stale packages remain unbuilt" (`:95`).
   - D6 starts a new `lym gate testmap` on any tick where "the last refresh reported stale packages left unbuilt" (`:103`).
   - So one dirty or untracked Go file in the prime worktree makes the dobby spawn a refresh, holding a gate slot, on every tick indefinitely.
   - This contradicts D6's own bound that "a package skipped as dirty waits for the next prime HEAD change" (`:111`), and the D6 acceptance criterion (`:188`) encodes the loop.
   - A lock-held skip, for example from a dobby restart that orphaned an in-flight call, feeds the same loop.
   - Fix: D5's count leaves out skipped packages, or reports them apart, and D6 retriggers only on built-but-capped leftovers.

2. **MEDIUM, consistency. D4/D5 staleness × D5 dirty skip, `decision-record.md:85`, `:90`, `:186`.** Untracked files make a package stale and also guarantee it is skipped.
   - D5 counts "a Go file in its directory, tracked or untracked" as making a package stale (`:85`).
   - It also skips any package with "a tracked or untracked Go file that differs from HEAD" (`:90`). An untracked file is always absent from HEAD, so a package that is stale because of an untracked file is always skipped and never rebuilt.
   - The D5 acceptance criterion "rebuilds only packages with ... a changed or untracked file" (`:186`) cannot pass alongside the D5 criterion "skips a package whose files differ from HEAD" (`:187`).
   - Fix: decide which one wins, and state it once in D5 and its criteria.

3. **LOW, consistency. D7 red report × D8 match, `decision-record.md:131`, `:127`, `:143`.** A red-gate report holds none of the fields D8's match test reads.
   - On a red tier, D7 "writes a report saying the gate was red and judges nothing" (`:131`). That report records no package hashes and no step-1 test hashes, which are D7 step 6's fields (`:127`).
   - D8's match test ("every judged package's Go files hash the same, and step 1 ... gives the same tests") is vacuous or undefined over it (`:143`).
   - The record does not say whether Code-Burler-Review then sees "no findings" or the "no current report" line.
   - The case is reachable: the `code-burler-fix` key's red attempt overwrites an earlier green `webster` report in the same gate directory.
   - Fix: state that a red report never matches, or that it records the step-1 tests.

4. **LOW, consistency. D7 step 2 × steps 3 and 4 × D1, `decision-record.md:121-124`.** Step 2 drops tests by results that only exist after step 4 runs them.
   - Step 2 drops "tests D1's eligibility rules leave unjudged" before the cap in step 3 and the runs in step 4.
   - D1's rules (`:32`) include skipped, failed and covers-nothing, which are only known once step 4 has run the test.
   - So the step-2 drop and the step-3 cap of `redundancy_max_tests` cannot be applied in the stated order: a test that later turns out to be skipped or empty still uses up a cap slot.
   - Fix: split the static drops (out-of-process scan, tier, keep) into step 2 and move the run-result drops after step 4.

Tool calls: none; the inherited context covered every pair.
