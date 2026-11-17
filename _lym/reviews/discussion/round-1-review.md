---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md:78
    summary: D4's memory_mb quotient has no floor, so a tier whose memory_mb is below redundancy_test_memory_mb gets a side-by-side count of 0 and the run stalls, while a floor of 1 breaks the claim that the tier's cap bounds the processes together
    fix: In D4, state the rule when a mapped tier's memory_mb is below redundancy_test_memory_mb. Recommended - map mode refuses that hub naming the tier, both values and the way forward (raise memory_mb or lower redundancy_test_memory_mb with lym config gate), so the bound stays true; add that refusal to D1's refusal list, to the PATTERN-refusal-way-forward constraint and as a D4 acceptance criterion. If instead the count floors at 1, say so and rewrite the sentence ending "so the tier's cap bounds the run's test processes together" into a Bound naming that one test may exceed memory_mb.
    origin: lens:bound-breaker
  - id: F2
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md:66
    summary: D3 builds lym once for tier 1 tests, but the only reader of LYM_PREBUILT_LYM is internal/testkit/lymbin and the spawn scan marks every test reaching lymbin out of process, so no test D3 runs reaches that build and the D3 criterion on line 179 cannot be met
    fix: In D3, delete the sentence that opens with When the call measures tier 1 tests and builds lym once for LYM_PREBUILT_LYM, and state instead that map mode builds no lym, since every test that reaches lymbin is spawn-marked and not run (evidence internal/testcover/spawnscan.go ScanSpawns doc, internal/testkit/lymbin/lymbin.go prebuilt). Delete the D3 acceptance criterion on line 179 that a tier 1 test of a map run reaches the lym the run built once.
    origin: lens:walkthrough
  - id: F3
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:75
    summary: D4 bounds side-by-side processes per tier but never says whether a package's tier 0 and tier 1 tests run at once, so up to p0 plus p1 processes may run with neither tier's parallel nor memory_mb bounding the total, and the coverage builds' -p names no tier
    fix: In D4, state one rule for a package holding both tiers' tests. Recommended - the package's tier 0 tests run first, then its tier 1 tests, each tier side by side up to its own clamped count, so at most one tier's count runs at once. In D4's Bound, name which tier's parallel sets the coverage builds' -p (recommended tier 0's, as today's holder does). Keep the D4 acceptance criterion and make it say the count never passes the running tier's parallel.
    origin: lens:interactions
  - id: F4
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:98
    summary: D6's check before the coverage build cannot cover linked packages, whose set the build's own go list step produces, and an edit made during the build and reverted before the pre-write check still reaches the map under HEAD's hashes; the record drops the bound today's code states and claims committed files only
    fix: In D6, replace "Before a package's coverage build and again before its map write" with the order the build forces - once the build has listed the linked packages and compiled, and again before the map write. Add a Bound line - an edit made after one check and reverted before the next goes unseen, so the map holds coverage of the edited code under the committed hashes. Reword the Rationale so it no longer claims a mid-run edit cannot reach a map.
    origin: lens:bound-breaker
  - id: F5
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:49
    summary: D1's Bound says the refused flags are the only gate features map mode skips, yet --stop-on-fail, --dir/-C and --main-packages are neither refused nor said to apply, and map mode also skips each tier's command, filter and shared build step
    fix: In D1, add --stop-on-fail and --dir/-C to the refused flags, each naming the same way forward, and state that --main-packages applies with --card as for any call. Rewrite the Bound to name what map mode skips - the tiers' commands, filters, memory_mb scopes and the shared build step, plus lint, report, guard step, carry and card fixer - and what bounds that - the run is on command and writes only maps. Extend the D1 refusal criterion to list --stop-on-fail and --dir.
    origin: lens:bound-breaker
  - id: F6
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:39
    summary: D1's package set holds only packages with a tier 0 or tier 1 test, so a package whose last mapped test was deleted is never re-measured and keeps a map whose stale entries lym gate redundant and the report step still count as covers
    fix: In D1, widen the package set to every package of the scope that has a map or holds a tier 0 or tier 1 test, and say that a package with no mapped test gets a map of its file hashes with no tests, as today. Add an acceptance criterion - D1 or D2, a map run over a package whose last mapped test was deleted rewrites its map with no test entries.
    origin: lens:interactions
  - id: F7
    severity: LOW
    class: design
    location: _lym/discussion/decision-record.md:137
    summary: D9 retargets only the missing-map refusal of lym gate redundant, but the unreadable-map and failed-build-only refusals share the same way-forward text naming lym gate testmap
    fix: In D9, change the sentence to say every refusal of lym gate redundant that names a map rebuild - missing, unreadable, or recording only a failed build - names lym gate test --map <package> from the worktree. Extend the D9 refusal criterion to the failed-build-only map.
    origin: lens:existing-paths
  - id: F8
    severity: LOW
    class: consistency
    location: _lym/discussion/decision-record.md:99
    summary: D6 and its criterion check any file under the package's directory, which takes in nested packages' files, while a package owns files only down to the next package directory or nested module
    fix: In D6, replace "no file under the package's directory or the directory of any in-module package it links" with "no file the package or any in-module package it links owns, a package owning the files under its directory down to the next package directory or nested module". Make the D6 criterion on line 183 say "a modified file the package or a linked package owns".
    origin: lens:walkthrough
  - id: F9
    severity: LOW
    class: consistency
    location: _lym/discussion/decision-record.md:114
    summary: D7 says the run still writes every map it measured, which contradicts D6 for a package measured and then found dirty or under a moved HEAD before its write
    fix: In D7, change "it still writes every map it measured, failed entries included" to "it still writes the map of every package D6 lets through, failed entries included".
    origin: lens:interactions
  - id: F10
    severity: LOW
    class: consistency
    location: _lym/discussion/decision-record.md:144
    summary: The PATTERN-gate-slot constraint calls a map run a gate test run, but that entry's background allows a gate test only as a tier run holding its tier's slots or as a one-slot non-tier holder, and a map run holding the larger of two tiers' asks is neither
    fix: In the PATTERN-gate-slot constraint, add that the background's slot rule names the map run of lym gate test --map as holding the larger of tier 0's and tier 1's slot asks, and add pattern/PATTERN-gate-slot.md to Scope's list of backgrounds and to the closing Review acceptance criterion.
    origin: lens:existing-paths
  - id: F11
    severity: LOW
    class: design
    location: _lym/discussion/decision-record.md:94
    summary: D5's list of tests that passed only alone lives only in the call's output, while their entries enter the map as plain passes that lym gate redundant and the report step count as covers, so a real flake is masked once the output is gone
    fix: In D5, add a Bound line - a test that passed only alone enters the map as a plain pass, and the list in the call's output and log is the only trace, so a flaky test can stand as a cover until its collision is fixed.
    origin: lens:bound-breaker
---

### [BLOCKING] D4's memory_mb clamp has no floor

**Location:** `_lym/discussion/decision-record.md:78`

**Issue:** `memory_mb` accepts any value from 1, and the template's `redundancy_test_memory_mb` is 2048 (`internal/gateslot/template.yaml:7`).
A tier with `memory_mb: 1024` gives a count of 0.
At 0, `runSideBySide` starts no worker and its feeder blocks on `indexes <- i` (`internal/gatecli/testmappackage.go:386-399`).
At a floor of 1, the floor `clampParallelToMemory` keeps today (`internal/gatecli/redundantmutants.go:539`), one 2048 MB test exceeds the tier's 1024 MB, which falsifies D4's own claim on line 79.
The record states neither case.

### [BLOCKING] D3's prebuilt lym reaches no test it runs

**Location:** `_lym/discussion/decision-record.md:66`

**Issue:** The only reader of `LYM_PREBUILT_LYM` is `internal/testkit/lymbin` (`lymbin.go:50`).
`testcover.ScanSpawns` marks every test that references `lymbin`, directly or through a followed call, as out of process (`internal/testcover/spawnscan.go:170,355`).
D3 runs no spawn-marked test.
The build costs a `lym` build per call and reaches nothing, and the criterion on line 179 describes no test that can run.
Today's verb does the same wasted build, so "as a tier run does" carries an existing defect forward as a premise.

### [MEDIUM] D4 leaves the two tiers' concurrency open

**Location:** `_lym/discussion/decision-record.md:75`

**Issue:** D1 measures both tiers in one pass, and D4 bounds the count per tier.
On the `_bolt` hub, where both tiers set `parallel: 4`, running them at once gives 8 processes of up to 2048 MB each.
Neither tier's `parallel` nor its `memory_mb` bounds that total, so line 83's "the same total as a scope around them" fails.
D4's Bound says the builds are bounded by "the tier's `-p`" without naming the tier.
Today's code takes one count from tier 0 alone (`testmappackage.go:604-608`).

### [MEDIUM] D6's pre-build check and its dropped bound

**Location:** `_lym/discussion/decision-record.md:98`

**Issue:** `startBuild` lists the linked packages and compiles in one goroutine (`testmappackage.go:224-236`).
D2's one-ahead build starts during the previous package's tests (`:187-188`), so no check before the build knows which linked directories to read.
Today's code checks after the build is taken up (`:172`) and again after the tests (`endJob`).
`internal/gatecli/doc.go:1133-1134` states the bound that an edit made and reverted between reads goes unseen.
D6 drops that bound and claims a map is written only from committed files.
That leaves the check's order for Plan to work out, a deferred fact the design rests on.

### [MEDIUM] D1's Bound misstates what map mode skips

**Location:** `_lym/discussion/decision-record.md:49`

**Issue:** `lym gate test` also takes `--stop-on-fail`, `--dir`/`-C` and `--main-packages` (`internal/gatecli/cli.go:119-130`).
D1 neither refuses them nor says what they do under `--map`.
Map mode also never runs a tier's command, its `filter` or the shared build step.
The Bound claims the refused flags are the only skipped features, which is false, and the attack-surface rule wants each new routing edge bounded.

### [MEDIUM] D1's package set misses a package whose tests were deleted

**Location:** `_lym/discussion/decision-record.md:39`

**Issue:** The set is "the packages holding a test of either tier".
A package whose last mapped test is deleted falls out of every map call, so its map is never replaced.
Its entries name tests that no longer exist, and `lym gate redundant` and the report step read them as covers.
Today a package with no mapped test gets a map of file hashes and no tests (`testmappackage.go:131-132`).

### [LOW] D9 retargets one of three refusals sharing one way forward

**Location:** `_lym/discussion/decision-record.md:137`

**Issue:** `internal/gatecli/redundant.go:162-174` builds one `rebuild` string naming `lym gate testmap` and uses it for the unreadable, missing and failed-build-only map refusals.
D9 and its criterion name only the missing-map one.

### [LOW] D6 counts nested packages' files as the package's

**Location:** `_lym/discussion/decision-record.md:99`

**Issue:** A package owns files only down to the next package directory or nested module (`internal/gatecli/doc.go:1071-1072`).
"Any file under the package's directory" makes a dirty file in `internal/x/sub/` mark `internal/x` unmeasured.

### [LOW] D7 contradicts D6 on maps written

**Location:** `_lym/discussion/decision-record.md:114`

**Issue:** D6 writes no map for a package found dirty, or under a moved HEAD, before its write, although its tests ran.
D7's "still writes every map it measured" says the opposite for that package.

### [LOW] PATTERN-gate-slot's background does not admit a map run

**Location:** `_lym/discussion/decision-record.md:144`

**Issue:** `pattern/PATTERN-gate-slot.md:12-16` allows a gate build or test only inside a tier run holding its tier's slots, or in a one-slot non-tier holder.
A map run holding the larger of two tiers' asks is neither.
The record removes the testmap holder from `internal/gateslot`'s doc but prescribes no edit to this background.

### [LOW] D5's rerun list leaves no trace in the map

**Location:** `_lym/discussion/decision-record.md:94`

**Issue:** D7 writes no run record, and a test that passed only alone enters the map as a plain pass.
Once the call's output is gone, a real flake stands as a cover and nothing marks it.
The bound needs saying.
The map format is out of scope, so the fix states the bound rather than adding a marker.

## Rejected

- Focus 1, PATTERN changes: neither constraint changes an entry's overview line; both edit a background only. The spec's `## PATTERN changes` bullets prescribe overview lines, so no bullet is owed, and "the entry's overview line does not change" states a fact rather than a change.
- Focus 6, relocation: D2's study figures and D5's `tmuxkit.Main` collision are the evidence their decisions rest on, which the spec requires in the record. The D4 and D7 Rationale and Bound lines carry bounds the attack-surface rule requires. None of these is deliberation narrative to move into `support-log.md`.
- D1 detached keying and the card fixer: `testRequest.words` is the one place the key is hashed, and `handsOff` needs `--gate`, which map mode refuses. Both hold.
- D7's slot hold: `slots: all` is the tier's zero value, and D7's Bound covers it.
