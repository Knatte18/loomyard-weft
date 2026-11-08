---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:81
    summary: D5 states no outcome for a package with no test binary to build or a failed coverage build, so under its no-map stale rule, path order and the cap, D6 restarts a refresh on every tick and later packages never get a map
    fix: In D5, add that a package whose coverage build fails, or which has no test file or no tier0 or tier1 test, gets a map recording its Go file hashes and no tests (with the build failure as its cause where one occurred), so it is stale again only when its files change or its map passes test_map_max_age_days; state that such a package counts against --max. Add a D5 acceptance criterion that a package with no test file and one whose coverage build fails each get a map and are not rebuilt on the next call while their files are unchanged.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:119
    summary: D6 says a dobby stop ends the test-map child because runproc.EndRole ends every descendant, but EndRole's contract spares every record of another role under the owner and the processes below it, and the child is registered under HubOwner beside the prime dobby
    fix: In D6, replace the sentence about runproc.EndRole ending every descendant with a stop route that holds under EndRole's contract, namely that the prime dobby's stop also calls runproc.EndRole for runproc.HubOwner with runproc.RoleTestMap (new), and keep the acceptance criterion that a dobby stop ends a running refresh.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:153
    summary: The report records function hashes only for judged tests while D8's match and the repeat skip compare step 1's whole set, so any diff touching a dropped test (tier2, keep-marked, undeclared tier, past the cap) never matches
    fix: In D7 step 6, record the function hash of every test step 1 returns, judged or not, and the Go file hashes of every package holding one; in D8, match HEAD's step 1 set against that recorded set. Extend the D7 repeat-call criterion and the D8 criterion with a diff that also changes a tier2 test.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:106
    summary: D5's time bound, which D6's orphan bound and D7's never-changes-the-result rule inherit, omits the gate-slot wait and puts no limit on a coverage or lym build, and a timed-out test's descendant holding its output pipe can stall the wait
    fix: In D5 and D7 step 4, state that a timed-out test is ended with its whole process group and its wait is bounded, and that each coverage build and the lym build run under a stated timeout (reuse the tier's timeout_min or name a new key with a default). Restate D5's time bound to include the slot wait as unbounded by the verb, and narrow D6's orphan sentence to the bound D5 then states.
  - id: F5
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:142
    summary: D7 step 1 compares tests file by file, so a renamed or moved test file makes every unchanged test in it absent at the merge base and fills redundancy_max_tests ahead of the new tests; a deleted test file has no current text and no stated outcome
    fix: In D7 step 1, look up each current top-level test by package and name across the package's _test.go files at the merge base, take it when that text differs or no such test exists, and pass over a deleted _test.go path. Add the rename case to the D7 merge-base criterion.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:62
    summary: D3's bound says a duplicate-text insertion costs at most one wrong hint, but the insertion shifts the ordinal of every later identical block in the function, so several keys can match wrong blocks and a false contained verdict follows
    fix: In D3's Bound, say that inserting a block whose text repeats one in the same function shifts the ordinal of every later identical block, so each judged test covering one of those blocks can get a wrong cover, and that review reading the finding bounds it; drop at most one wrong hint.
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:107
    summary: D5's bound names only dependency changes outside the package directory, but non-Go files inside it (testdata, embedded files) are hashed neither for staleness nor for the mid-package landing recheck
    fix: In D5's Bound, replace A dependency change outside the package directory with A change to a dependency or to a non-Go file of the package directory, such as testdata or an embedded file, and keep the age-limit clause.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:174
    summary: D7's false-cover bound names only an edit in the diff, while the same false cover follows from changes landed since the map was built, package-level test tables, TestMain and testdata, and the report names the map's age only for a package whose files differ
    fix: In D7's Bound, say a map test's reach can change through any code outside its own function text, from the diff or landed since the map was built; in D7 step 6, name the map commit and age for every judged package, not only one whose files differ, and adjust the matching D7 criterion.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:146
    summary: D1 moves the eligibility rules into internal/testcover because a main package cannot be imported, yet D1 and D7 step 2 still place the spawn scan in cmd/testtiming/spawnscan.go, which the gate step cannot call
    fix: In D1, state that the static spawn scan moves from cmd/testtiming/spawnscan.go into internal/testcover with the eligibility rules, and in D7 step 2 cite the scan as internal/testcover's.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:237
    summary: The criterion says a test covering a block no map test covers is reported unique, but step 5 counts other judged tests as covers, so a second judged test covering that block can make it contained
    fix: In the D7 acceptance criterion, read a test covering a block of an edited function that no map test and no other judged test covers is reported unique.
---

The handler's own review reached F1, F2, F4, F5, F7 and F8 independently before the forks returned; the origin key names the lens whose report each kept finding is drawn from.

### [BLOCKING] D5 leaves a package with nothing to build without a map, and D6 then loops

**Location:** `_lym/discussion/decision-record.md:81` (D5 staleness, `--max`), `:112` (D6 retrigger on capped leftovers)

**Issue:** D5 names outcomes for dirty, locked, landing-changed and timed-out cases, but none for a package whose coverage build fails or yields no binary.
`go test -c` builds nothing for a package with no test file, and this worktree has eight such package directories under `internal/` alone (`internal/hibernateconfig`, `internal/lymdirs`, `internal/patternlint`, `internal/planindex`, four under `internal/testkit/`), plus `contracts/` ones the bound-breaker found.
With no map written, "it has no map" keeps each stale for good, and nothing exempts it from `--max`.
Once the rest are mapped, every refresh spends its five-package batch on the first of these in path order and reports the others as capped, so D6 starts a refresh on every tick, each holding a gate slot, and a changed package sorting after them is never rebuilt.
D6's "Packages a refresh skipped never start another" does not cover them, since they are not skipped.
Interactions, walkthrough and bound-breaker each raised this; it is kept at BLOCKING because the record's own stated rules produce the loop on this repository as it stands.

### [MEDIUM] D6's dobby-stop claim contradicts `runproc.EndRole`'s contract

**Location:** `_lym/discussion/decision-record.md:117` to `:119`, criterion `:235`

**Issue:** The prime dobby is registered under `runproc.HubOwner` as `dobby` (`internal/beamcli/dobbydeps.go:77`) and stopped through `EndRole(..., owner, runproc.RoleDobby, ...)` (`internal/orchcli/cli.go:276`).
`EndRole`'s doc says every record of another role under the owner, and every process below it, is left alone (`internal/runproc/end.go:72`), and `ender.end` marks the `test-map` record spared.
I read `planRecords` (`internal/runproc/end.go:355` to `:381`): its descendant walk does not consult `skip`, so today the child is still signalled as the dobby's descendant.
The criterion would pass by that undocumented path, which is why this is MEDIUM and not the BLOCKING two lenses gave it: the premise D6 cites is false by the module's contract, and code brought into line with its doc breaks the stop.
Round 1's fixer self-check read the same walk and took it as the guarantee; the walk's behaviour is real, but `EndRole`'s documented contract says the opposite, so the stop should not rest on it.

### [MEDIUM] The report's recorded set and D8's match set differ

**Location:** `_lym/discussion/decision-record.md:153`, `:182`, `:136`

**Issue:** Step 6 records a function hash "per judged test"; D8 matches when step 1 on HEAD "gives the same tests with the same function hashes", and the repeat skip uses D8's match.
Step 1 returns every changed test, including those steps 2 to 4 drop, so a diff that also changes a `tier2` or keep-marked test never matches: Code-Burler-Review always reads the no-report line and every repeat gate call rebuilds and reruns.

### [MEDIUM] D5's time bound does not bound the run

**Location:** `_lym/discussion/decision-record.md:106`, `:121`, `:128`, `:149`, `:159`

**Issue:** The bound counts one `lym` build, one coverage build per package and per-test timeouts.
It leaves out the wait for the gate slot D5 holds, and neither build has a timer, so a hung build has no limit.
A timed-out `tier1` test can leave a `git` or `lym` grandchild holding the inherited output pipe; waiting on the test process then blocks until that grandchild exits, which no stated term bounds.
D6's orphan sentence inherits the gap, and in D7 the same hang stalls the `lym gate test` call, so its caller sees a hang rather than the tiers' result.
Merges bound-breaker F3 and F5 and interactions F4.

### [LOW] D7 step 1 misreads renamed, moved and deleted test files

**Location:** `_lym/discussion/decision-record.md:141` to `:147`

**Issue:** `impactset.ChangedPaths` lists a renamed file by both paths and lists deleted files, without status (`internal/impactset/targets.go:99`).
A renamed file's tests are all absent at the merge base under its new path, so they are judged as new and fill `redundancy_max_tests` in file then name order ahead of the diff's new tests.
Merges bound-breaker F7 and existing-paths F3; LOW because renames of large test files are rare.

### [LOW] D3's ordinal bound understates the cost

**Location:** `_lym/discussion/decision-record.md:60` to `:62`

**Issue:** An inserted block repeating the text of others in the same function shifts every later identical block's ordinal, so more than one key can match a wrong block, and the error runs towards a false contained verdict.
Identical error-return blocks are common in Go.
Bound-breaker rated it MEDIUM; LOW here, since every verdict is review evidence and the record already names the mechanism.

### [LOW] D5's bound misses non-Go files in the package directory

**Location:** `_lym/discussion/decision-record.md:107`, `:70`, `:94`

**Issue:** The map hashes Go files only, so a change to `testdata/` or an embedded file such as `internal/gateslot/template.yaml` neither marks the map stale nor trips the mid-package landing recheck; the bound names only changes outside the directory.

### [LOW] D7's false-cover bound is narrowed to the diff

**Location:** `_lym/discussion/decision-record.md:173` to `:175`, `:156`

**Issue:** A map test can stop reaching a block through landed dependency changes since the map was built, package-level tables, `TestMain` or `testdata`, none of them in the diff.
The report names the map's age only when the package's files differ, so a week-old map behind a matching package shows Code-Review no drift.

### [LOW] The spawn scan's home is unstated after D1's move

**Location:** `_lym/discussion/decision-record.md:32`, `:34`, `:146`

**Issue:** D1's rationale is that a `main` package cannot be imported, yet both D1 and D7 step 2 cite the scan at `cmd/testtiming/spawnscan.go`, which the gate step cannot call.
Interactions rated it MEDIUM; LOW here, since "the eligibility rules move with the engine" already implies the move and only the citation lags.

### [LOW] A D7 criterion contradicts step 5

**Location:** `_lym/discussion/decision-record.md:237`, `:151` to `:152`

**Issue:** Step 5 counts other judged tests as covers, so two judged tests sharing a block no map test covers leave one contained, while the criterion says such a test is reported unique.

## Rejected

- Existing-paths F2, a caught signal during the step turning a green call into a non-zero exit: a signal is the caller cancelling the call, not an error the step meets, and the same holds during the tiers; the longer exposure window is F4's subject.
- Bound-breaker F8, a red lint with green tiers still judging: a lint failure does not touch the coverage evidence, and the commit that clears it changes the files, so D8's hash match drops that report.

## Focus verdicts

- Config route (focus 1): a route exists and is not a defect. `gates` is a per-name map, not an open map (`internal/gateslot/config.go:99`, `:105`), and `yamlengine.FillMissing` steps into each `gates` entry key by key (`internal/yamlengine/fill.go`, `fillMapping`; `internal/yamlengine/doc.go` on per-name maps), so the hub base's `webster`, `parent-align`, `darn-write` and `code-burler-fix` entries load with the template's `reports`. `reports: []` is present and never replaced, which bounds it per key; `gates` stays closed to the local layer as a guard key.
- Merge-base text (focus 2): D7 step 1 reads the merge base through `parentMergeBase`, which reads the warp's `Fabric.MergeBase`, and the text through `fabricengine.CodeFileAt` (new) on the warp; no parent branch judges nothing, and an unreadable merge base is a report line. The one gap is F5.
- D5 and D6 skips (focus 3): a dirty or locked package beyond the cap can cause at most one refresh that builds nothing, which then reports no capped leftover, so no loop; the untracked-file rule is one rule (tracked set for staleness, any untracked Go file skips). The loop the focus points at comes instead from packages with nothing to build, F1.
- Landing during a refresh (focus 4): `Fabric.Merge` syncs and runs `MergeStart` in the prime's warp checkout under the weft write lock (`internal/fabricengine/merge.go`); D5's dirt check runs with `GIT_OPTIONAL_LOCKS=0` (`internal/gitexec/gitexec.go:243`), so it never takes the index lock, and the per-package hash recheck covers the package's Go files. Gaps: non-Go files (F7) and the time bound (F4).
- D7's bound claims (focus 5): F8 for the false-cover edits, F4 for the hang the never-changes-the-result rule does not cover, and the signal case rejected above.
