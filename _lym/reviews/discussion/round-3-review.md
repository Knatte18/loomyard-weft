---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: "_lym/discussion/decision-record.md:65-71, 213-214, 217-219, 222-225, 239; _lym/discussion/support-log.md:62, 115-116"
    summary: "The record still carries D2's staleness rule and both questions for the parent although the two Added after Discussion entries answer them"
    fix: "In decision-record.md make these edits. (1) D2: delete the sentence at lines 65-66 that starts 'testmap counts a map stale, besides today's rules' and ends 'whether or not a test covered a block of that package'; delete the Rationale clause at lines 70-71 that starts 'and a map whose dependency drift testmap never repairs'; keep the marker, the per-package file-set record and the repeated dirty check. (2) D5: add one sentence to its closing Bound paragraph: 'A dependency-drift miss lasts until testmap rebuilds the map by its present rules: the package's own files changed at HEAD, the age limit, or a failed build at another HEAD.' (3) Acceptance criteria, the D2 stale criterion at line 239: replace it with 'D2: testmap leaves a map as fresh when only a linked package's file changed at HEAD and the map's own files and age are unchanged'. (4) Auto-mode assumptions: cut the clause 'and testmap rebuilding maps whose linked dependencies drifted'. (5) Open risks: delete the first bullet (lines 217-219) and replace the third bullet (lines 222-225) with the statement the Q11 entry prescribes: coverage narrows only while a map is fresh; the share of fresh maps is set by the cadence note testmap-upkeep governs, and by the measured rate under Constraints landings to internal/fabricengine still miss about twice a day; the tasks raddle files from this record must be ordered after the task that builds testmap-upkeep's design. Keep the second bullet. In support-log.md: reword the Rejected alternatives bullet at line 62 so it records the rejected course as 'raise testmap's staleness for dependency drift in this task: the parent put the staleness policy with note testmap-upkeep'; mark Q10 and Q11 at lines 115-116 resolved by the parent's Added after Discussion entries; add to the Round 2 list that F4 (decision) was answered and applied in round 3."
  - id: F2
    severity: BLOCKING
    class: design
    location: "_lym/discussion/decision-record.md:78-88, 153-157; internal/testcover/testfuncs.go:28-30; internal/gatecli/tierrun.go:387-417"
    summary: "D3's union can name a test no binary on the gate's host compiles, so the selected invocation runs zero tests in that package and emptyTierFailure fails the gate"
    fix: "In D3 define the union's second member as 'every test of the tier declared in a test file of the package that build constraints admit on the gate's host, read as holdsTierTest in internal/gatecli/tierrun.go reads them (go/build Context.MatchFile with the tier's tag), that its map does not record'; a package whose union is then empty counts as none selected, as D3 already says. In D6 state that the selected invocation's -run argument sits in the run's own args and not in the call's args, because a filter among the call's args exempts the whole tier run from the empty-tier check. Add an acceptance criterion under D3: 'an importing package whose only unrecorded declared test sits in a test file constrained to another GOOS runs no test, and the gate passes'."
  - id: F3
    severity: MEDIUM
    class: design
    location: "_lym/discussion/decision-record.md:59-64; internal/gatecli/testmap.go:270-277, 363, 414-422; internal/gatecli/testmappackage.go:57-62, 91-98"
    summary: "D2's repeated dirty check does not say it re-reads git status, and an edit made and reverted during a package's build and tests ends up under the committed hashes with no stated bound"
    fix: "In D2 replace 'testmap repeats its dirty check over every linked package's files' with: both dirty checks of a package are fresh reads of the worktree status, the first taken immediately before the package's build and the second after its last test, each over every linked package's files, and neither reuses the list testmap read before its walk. Add a Bound to D2: an edit to a linked file made after the first read and reverted before the second goes undetected; the map then holds blocks measured on the edited code under the committed hashes, and only the landing gate catches a test that this wrongly leaves out. Replace 'A map's recorded hashes are therefore the content at its Commit' with 'A map's recorded hashes are the content at its Commit; the Bound below names the one window in which the measured blocks can differ'. Add an acceptance criterion: 'D2: a linked package's file dirtied after the walk started and still dirty after a package's tests leaves that package without a map'."
  - id: F4
    severity: MEDIUM
    class: design
    location: "_lym/discussion/decision-record.md:139-141; internal/gateslot/template.yaml:45-47; contracts/stencils/stencils.go:81"
    summary: "D5 lets inert_paths exempt a linked package's recorded non-Go file, so an edit to an embedded stencil in a hub with a markdown inert glob narrows importers to nothing selected, with no stated bound"
    fix: "In D5's last bullet add that the inert_paths exemption never covers a file that a go:embed pattern in the linked package's Go files names; the scope reads those patterns with go/parser from the recorded Go files, and a directory pattern names every file below it. Add one acceptance criterion under D5: 'an edit to an embedded file of A that inert_paths matches is a miss'. Alternatively drop the exemption and state in the Bound that a hub's package-directory markdown then misses its importers."
  - id: F5
    severity: LOW
    class: design
    location: "_lym/discussion/decision-record.md:142-148, 260"
    summary: "D5 says a plain added declaration is reached only through changed code, but an added package-level name equal to a predeclared identifier rebinds unchanged code of its own package"
    fix: "In D5's miss list add: an added top-level declaration of a linked package whose name is a predeclared identifier of the universe scope, such as max, min, clear, new, len, error or string, counts as a change outside function declarations. Replace the dot-import clause of the Bound (lines 147-148) with nothing, since that case is then a miss, and add 'an added function named max in A' to the miss examples in the D5 acceptance criterion at line 260."
---

### [BLOCKING] The record contradicts the parent's recorded decisions

**Location:** `_lym/discussion/decision-record.md` D2 (staleness sentence and Rationale), Auto-mode assumptions, Open risks, the D2 stale acceptance bullet; `_lym/discussion/support-log.md` Rejected alternatives and Question ledger.

**Issue:** The first `### Added after Discussion` entry decides that the writer removes D2's staleness rule, and states in D5 that a dependency-drift miss lasts until testmap rebuilds the map by its present rules.
The second decides that Open risks states the cadence dependency and the raddle ordering instead of asking the parent.
Neither is applied.
D2 still adds the "counts a map stale" rule, its Rationale still argues for it, the Auto-mode assumption still accepts its cost, the first Open risks bullet still asks the parent whether D2 may change that policy, and the third still asks whether to land before the cadence is set.
The rubric calls such a line a `consistency` finding whose fix drops it.
A plan writer reading this record would plan the testmap staleness change that the parent refused.
The parent's decision wins over the writer's text.

### [BLOCKING] D3's union can leave a selected package with zero tests and fail the gate

**Location:** `_lym/discussion/decision-record.md` D3 (union, second member) and D6; `internal/testcover/testfuncs.go` `ScanTestFuncs`; `internal/gatecli/tierrun.go` `emptyTierFailure`, `holdsTierTest`.

**Issue:** The round-2 directive asked whether the union can hold tests outside the tier or tests the watch skip removes.
D3 now closes both: it keeps only tests of the tier and drops watch-skipped names.
It does not close a third route to zero tests.
The second union member is every test of the tier "the package declares at the working tree that its map does not record".
`ScanTestFuncs` documents that build constraints are not read, so a test in a file only another platform compiles counts as declared.
testmap lists the same tests as mapped, runs each alone under a binary that does not contain it, and leaves it out of the map.
Such files exist: `internal/githubclient/githubclient_windows_test.go` (`//go:build windows`) holds `TestWriteCachedToken_CreatesFileWithRestrictivePermissions`, and `internal/githubclient` imports `internal/proc` and `internal/pauserecord`, so it is an importing package.
On Linux the selected invocation's `-run '^(TestWriteCachedToken_CreatesFileWithRestrictivePermissions)$'` matches nothing there.
`goJSONCounter` then counts the package at zero, `holdsTierTest` finds the package's other tier tests in files that match, and `emptyTierFailure` reports the tier run as failed.
`emptytier_test.go` pins that a `-run` in the run's own args, the guard step's form, exempts nothing, and that only a filter among the call's args exempts the run.
So a gate with `scope: coverage` fails for such a package, and the fix is not available to the agent whose gate it is.
The acceptance criteria name the empty-tier case only for watch-marked guard tests.

### [MEDIUM] The dirty check is not stated fresh, and one race window stays unbounded

**Location:** `_lym/discussion/decision-record.md` D2, third paragraph; `internal/gatecli/testmap.go` `prepare`, `mapPackages`, `dirtyGoFile`; `internal/gatecli/testmappackage.go` `mapPackage`, `changedFiles`.

**Issue:** testmap reads the worktree status once in `prepare` into `r.dirty`, and `dirtyGoFile` consults that list for every package.
`changedFiles` compares file blobs at HEAD before and after the tests, so it sees a commit during the walk and no uncommitted edit.
D2 says testmap "repeats its dirty check over every linked package's files" after the tests.
Read literally against the present code, the repeat uses the same once-read list and detects nothing.
Read as a fresh status read, it still misses an edit made after the walk started and reverted before the check: the binary is compiled from the edited working tree, the profile measures the edited code, and the map records the committed hashes.
D2 then claims the hashes "are therefore the content at its Commit", which does not hold in that window, and states no bound.
Because narrowing trusts the map, a block profile measured on code other than the hashed code is a wrong record, not a widening.

### [MEDIUM] The inert_paths exemption narrows past an embedded file

**Location:** `_lym/discussion/decision-record.md` D5, last bullet; `internal/gateslot/template.yaml` `inert_paths` comment; `contracts/stencils/stencils.go` embed directives.

**Issue:** The directive's first question is whether an edit to an embedded non-Go file in a subdirectory, such as under `contracts/stencils/loom/`, can narrow importers to nothing selected.
D2's recorded set is right: it equals what `impactset.packageOfPath` maps, the nearest enclosing package directory, so a file under a subdirectory that is no package belongs to the package above it.
D5's last bullet then exempts a recorded non-Go file that `inert_paths` matches.
The template's own comment gives `*.md` as an example value, and the embedded stencils and specs are Markdown that tests read.
In a hub with that glob, an edit to an embedded stencil in a linked package is neither a function change nor a miss, so every importer selects nothing.
Under `diff`, a mixed diff still runs those importers, because `inert_paths` only empties a diff whose every file is inert.
The Decision introduces a bypass of the file-set guard and states no bound.
This hub's `gate.yaml` has no `inert_paths`, so the hole is for hubs that set a Markdown glob.

### [LOW] A predeclared name added at package level rebinds unchanged code

**Location:** `_lym/discussion/decision-record.md` D5, Rationale and Bound.

**Issue:** D5 exempts an added plain function, const, var without initializer and type without methods on the ground that unchanged code cannot reach them.
That holds for a fresh name.
It fails when the added name is a predeclared identifier: unchanged code of the same package that calls the builtin `max` or `min` resolves to an added package-level `max` or `min`.
The Bound names only the dot-import form of this case.
A miss rule on the added name closes both forms and costs nothing in practice, since the set of predeclared names is small and fixed.

## Rejected

- Directive point 1, the recorded file set: D2 matches `impactset.packageOfPath` for non-Go files, and the embedded-subdirectory case is recorded and compared. Not a defect; the inert exemption is F4.
- Directive point 2, packages with no covered block: D2 records every package of the coverpkg list, and D5's criteria name a changed const in a linked package no test covered a block of. Not a defect.
- Directive point 3, the non-body rate: Constraints state window, method and per-package counts. The per-package denominators (26, 12, 6, 9) reproduce from main's first-parent history; the total reproduces as 119 against the record's 118, a boundary difference the record's own "at least" wording absorbs. The added-declaration gap is F5.
- Directive point 4, the whole-declaration rule: D4 compares receiver, signature and body together; `funcKeyName` strips the pointer, so a value-to-pointer receiver change keeps the same key and is caught by the declaration compare and by D5's method-set miss. Not a defect.
- Directive point 5, directive lines: D5 makes every `//go:` line a miss wherever it sits. No `import "C"`, `// +build`, `//line` or `//export` line exists in the repository. Not a defect.
- Directive point 6, tier and watch skip: settled in D3. The platform route is F2.
- Directive point 7 is F1, point 8 is F3.
- The third Added after Discussion entry (the verify gate of commit 1b6e3fca stands): D6 already states that `--stop-on-fail`, the failed-test rerun and the verify budget act on both invocations, so the record agrees with it. Not a defect.
- Missing "Notes for the plan writer", rejected alternatives and call-site enumeration were not weighed, per the rubric.
