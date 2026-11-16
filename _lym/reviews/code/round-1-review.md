---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/runplan/fingerprint.go:163
    summary: PlanEditError changed webster's fingerprint-mismatch refusal from "since state.json was created" to "since the run state was created", so begin-batch, record-batch, recover-batch and validate now word it differently from run
    fix: Move the phrase out of runplan so webster keeps its text. In internal/runplan/fingerprint.go PlanEditError, format the tail as "...does not match this run's recorded fingerprint %s; %s" with wayForward() alone. In internal/websterengine/runplan.go PlanEditError, have the closure return "the plan changed since state.json was created; " + fingerprintMismatchWayForward(st, planDir, websterDir, stepRun). Update the PlanEditError doc comment to say the caller's text follows the fingerprints.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/websterengine/recoverbatch.go:688
    summary: No test reaches the way forward PersistRecoveryTerminal passes to runplan.CheckRecoveredSuspects; the suspect test calls an export_test.go wrapper that rebuilds the same argument itself
    fix: In internal/websterengine (recoverbatch.go or runplan.go), add an unexported production function recoveredSuspectReasons(geom Geometry, st *State, bs *BatchState, number int, head string) ([]string, error) that calls runplan.CheckRecoveredSuspects with planPathClause(fmt.Sprintf("\"lym webster recover-batch %02d\"", number)). Call it from PersistRecoveryTerminal at recoverbatch.go:688. In internal/websterengine/export_test.go, replace the body of the checkRecoveredSuspects wrapper with a call to recoveredSuspectReasons, so suspect_test.go's assertion on the reason text covers the production wiring.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/runplan/doc.go:16
    summary: The runplan package doc says nothing here names an implementer's verbs, but conflict.go writes and parses a plan-conflict header naming begin-batch and postbatch.go logs under the webster handle-bind tag
    fix: In internal/runplan/doc.go, replace "Nothing here names an implementer's verbs." with a statement that two texts keep webster's names because readers pin them, the conflict record's begin-batch header and the handle-bind log line, and that nothing else names an implementer or its verbs.
  - id: F4
    severity: LOW
    class: consistency
    origin: handler
    location: internal/websterengine/doc.go:1229
    summary: Webster's doc.go section "No shared substrate or parser with any other batch-implementation loop" now lists runplan's contents, which runplan's own doc.go already states, under a heading that the shared package contradicts
    fix: In internal/websterengine/doc.go, retitle the section to say webster shares only internal/runplan's run-plan machinery with other batch-implementation loops. Replace the three sentences that list runplan's contents ("They are the report contract..." through "...complete the set.") with one sentence saying webster reaches the shared batch machinery through internal/runplan. Keep the sentences about State embedding runplan.State and about webster passing its own way forward.
  - id: F5
    severity: NIT
    class: consistency
    origin: handler
    location: internal/websterengine/beginbatch.go:31
    summary: Two comments still name moved unexported helpers that no longer exist in websterengine, changedPlanFiles at line 31 and batchIdentity at line 288
    fix: In internal/websterengine/beginbatch.go, change "A changedPlanFiles error" at line 31 to "A runplan.ChangedPlanFiles error", and "by batchIdentity" at line 288 to "by runplan.BatchIdentity".
---

### [LOW] Webster's fingerprint-mismatch refusal text drifted

**Location:** `internal/runplan/fingerprint.go:163`; the old text is in `git show 194650f0:internal/websterengine/fingerprint.go:142`, and the unchanged sibling text is at `internal/websterengine/runlevel.go:575`.

**Issue:** The amended board entry says Webster's behaviour stays as it is.
The moved `PlanEditError` changed the clause "the plan changed since state.json was created" to "the plan changed since the run state was created".
The change description does not mention this change.
The `Run` entry check still renders the old clause, so one refusal now reads two ways depending on the verb that raised it.
The neutral wording is right for a shared package, so the Webster-specific phrase belongs in the text Webster passes.

### [LOW] No test reaches Webster's recover-batch plan-path way forward

**Location:** `internal/websterengine/recoverbatch.go:688`; the test is at `internal/websterengine/suspect_test.go:187`, through `internal/websterengine/export_test.go`.

**Issue:** D11 makes each shared refusal take its way forward from the caller.
The suspect-path reason's way forward is now an argument at the production call in `PersistRecoveryTerminal`.
The only test of that text calls the `export_test.go` wrapper `checkRecoveredSuspects`, which builds the `planPathClause` argument itself.
A wrong or empty argument at `recoverbatch.go:688` would therefore pass every test.

### [LOW] The runplan doc's claim that nothing names an implementer's verbs is false

**Location:** `internal/runplan/doc.go:16`; the counterexamples are at `internal/runplan/conflict.go:142,172` and `internal/runplan/postbatch.go:163,166`.

**Issue:** The change description deliberately keeps the conflict-record header (`begin-batch`) and the `webster: handle bind` log line, because tests and the record reader pin both.
The package doc still claims that nothing in the package names an implementer's verbs.
A Weaver author would take that claim at face value.

### [LOW] Webster's doc.go repeats runplan's contents under a heading the move contradicts

**Location:** `internal/websterengine/doc.go:1229`, in the section "No shared substrate or parser with any other batch-implementation loop".

**Issue:** The added paragraph lists what `internal/runplan` holds, which `internal/runplan/doc.go` already states.
That is a second copy, and it goes stale as runplan grows.
The section heading still says webster shares nothing with another batch-implementation loop, while the point of this diff is that webster's batch machinery is now shared.

### [NIT] Stale helper names in beginbatch.go comments

**Location:** `internal/websterengine/beginbatch.go:31` and `internal/websterengine/beginbatch.go:288`.

**Issue:** The comments name `changedPlanFiles` and `batchIdentity`, which moved to `runplan.ChangedPlanFiles` and `runplan.BatchIdentity`.
A rename search for the runplan names misses these lines.

## Rejected

- code-affected-set F1, PATTERN-cwd-resolution "Bound packages" lacks runplan: the code complies (runplan imports no `lymcwd`). Adding a package to a PATTERN list is a PATTERN change, and this round's directive declares none. That is a constraints-document question for the parent, not a defect the diff can fix within its declared changes.
- code-affected-set F3, the plan-conflict, mutate-lease and plan-baseline file names are not in `lymdirs`: PATTERN-lym-files reserves for `lymdirs` the naming of `_lym` and `.lym` in a path (`pattern/PATTERN-lym-files.md:41`). The runplan constants are file names inside a directory the caller passes. The decision record's Constraints line about the shared plan-conflict path is the writer's own claim about Weaver's later work, not the amended board entry, and the rubric forbids using it as the measuring stick.
- code-tests finding 2, the moved State JSON keys are pinned only by typed round-trips: this is coverage that existed before the diff. The diff copies the tags unchanged (`internal/runplan/state.go` against the removed block in `internal/websterengine/state.go`), and `internal/state`'s keeper walks embedded structs (`internal/state/keep.go:186-200`). The diff adds no behaviour here.

## Focus departures

- "Test economy: the two contained candidates in the redundancy report for runlevel_test.go and cli_test.go/parity_test.go, judged by comparing assertions with their covering tests."
  I did not compare these candidates' assertions with their covering tests, and I filed no finding on them.
  The diff's only edit to each of `TestGateParity_PlanGate`, `TestValidateCmd_ScopeFollowsRunProgress`, `TestRun_DeadEndRefusalSparesRecoverableBatches`, `TestRun_EntryValidationReachesMaster` and `TestRun_FreshRunOverNewGenerationAfterArchive` is the State-literal nesting the embed forces.
  The amended board entry requires Webster's existing tests to stay unchanged, so folding or deleting them falls outside this diff's scope.
