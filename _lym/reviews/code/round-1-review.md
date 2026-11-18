---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/loomcli/arm.go:69
    summary: the pr-findings exemption in resolveRunID, which keeps the findings file from being read as a run id, has no test row
    fix: "Add one row to the table of TestResolveRunID in internal/loomcli/arm_seed_test.go: verb pr-findings, args holding one file path such as .scratch/pr-findings.md, wantRunID shedrun.SelfRunID, no refusal. The existing rows for insert-card, remove-card and fmt are the model."
    origin: lens:code-tests
  - id: F2
    severity: LOW
    class: design
    location: internal/loomcli/prfindings.go:364
    summary: a crash between writing the friction note and updating the record makes the re-run write a second note for the same send-back, against D6's one note per send-back
    fix: "In writeSendBackNote, skip the write when the file frictionDir/pr-review-send-back-<n>.md already exists, since n is unique per send-back, so a re-run keeps the first note. Add one row to TestPRFindingsVerb that pre-creates that note and checks the friction directory still holds one file after the send-back."
    origin: lens:code-correctness
  - id: F3
    severity: LOW
    class: design
    location: internal/shedadapters/bouncer_amendment_test.go#TestBouncer_Amendment
    summary: no row covers a full review input over a changed subject, the path a send-back takes after PR-Review's fix-here commits, nor the review input note being none for an amendment record
    fix: "Add two rows to TestBouncer_Amendment: a settled round-1 record with the subject changed and a Full input written after the settle, expecting generation 2 with no Amendment mark and no base via requireFullGenerationOver; and an amendment generation record, expecting ReviewInputNote(runDir) to equal the none note."
    origin: lens:code-tests
  - id: F4
    severity: LOW
    class: design
    location: internal/driftengine/verify_test.go#TestDriftVerify
    summary: under diff command, a failing compare with an unruled file, and an exit 0 with every file already ruled, have no row; only exit 0 with an unruled file is pinned
    fix: "Add two subtests beside the command-mode loosened-check subtest in TestDriftVerify: a failing compare plus a widened file ruled uncaused, expecting a Stuck whose reason holds the does-not-match reason first and the uncaused reason after, and the stored file ruling; and a second call over the same range with exit 0, expecting Done with no new judge run and the stored loosened checks on the last check."
    origin: lens:code-tests
  - id: F5
    severity: LOW
    class: design
    location: internal/driftengine/loosened.go#loosenedTrailerLines
    summary: no test pins that only the last paragraph of each message counts, nor the sort, the duplicate removal or the CRLF handling, which D3's ruling key depends on
    fix: "Add a table-driven TestLoosenedTrailerLines in internal/driftengine with rows for a trailer in a middle paragraph being ignored, two messages with the same trailer yielding it once, unsorted trailers coming out sorted, and a CRLF message. Assert the exact returned slice."
    origin: lens:code-tests
  - id: F6
    severity: LOW
    class: design
    location: internal/loomcli/landingdeps.go#ShedStatus
    summary: the ShedStatus closure that maps a missing status file to an empty status has no test, and sendBackEdge has no row for a target row missing from the recipe or an entry gate that errors
    fix: "Add a test over pullRequestDepsGuardedBy that calls ShedStatus with no run directory and expects an empty Status and nil error, and with an unreadable status file and expects the error. Add two rows to TestPRFindingsVerb: edge producers whose Code-Bouncer target is absent, and one whose EntryGate returns an error, each expecting the read-failure refusal with its way forward."
    origin: lens:code-tests
  - id: F7
    severity: LOW
    class: consistency
    location: internal/landingshed/prreview_wait_test.go#TestPRReview_WaitSendBackHaltedOnADirtyTreeReroutesOnTheNextCallOnACleanTree
    summary: the redundancy report lists this test as contained, and its assertions are already made by the covering rows of TestPRReview_Wait and TestPRReview_Call
    fix: "Delete the test. TestPRReview_Wait row AWakeWhoseTreeIsDirtyAfterTheStopHaltsNamingTheFilesAndKeepsTheMark asserts the halt with no reroute and the mark kept, and TestPRReview_Call rows AMarkWithNoHistoryEntryOfTheRowsOwnKeepsTheMarkAndTheCountAndReroutes and ADirtyTreeHaltsNamingThePathsAndKeepsTheMark assert the reroute with the file as output on a clean tree and the mark kept."
    origin: handler
  - id: F8
    severity: LOW
    class: consistency
    location: internal/loomrecipe/recipe_test.go#TestRecipe_PRReviewReroutesToCodeBouncer
    summary: the redundancy report lists this test and TestRouting_ProgressAt as contained, but the PR-Review reroute edge of the darn and sampler recipes is asserted by no covering test
    fix: "Add a //testtiming:keep line directly above func TestRecipe_PRReviewReroutesToCodeBouncer, with the reason that it pins PR-Review's on_reroute edge in the tapestry, darn and sampler recipes, which the covering tests do not assert. Add the same directive above func TestRouting_ProgressAt in internal/loomrecipe/progress_test.go, with the reason that it pins the exact reroute table of the tapestry tail."
    origin: handler
  - id: F9
    severity: LOW
    class: scope
    location: docs/overview.md:1085
    summary: the Drift passages still say the judge rules only on expected changes and omit the loosened-check ruling and its Stuck, and the PR-Review passage still says the parent is asked once per pull request, though a send-back re-arms the ask
    fix: "In docs/overview.md, extend the Drift-Verify and Drift-Recheck lines near 1085 and 1328 with the judge's ruling on each loosened check in a test existing at the review base, caused or uncaused, an uncaused one making the row Stuck until a Loosened-Check trailer commit or a revert and lym loom resume. Reword the line near 1362 to say the ask is made once per pull request until a send-back clears it."
    origin: lens:code-affected-set
  - id: F10
    severity: NIT
    class: consistency
    location: internal/loomrecipe/testdata/tapestry-rows-before-motif.yaml:620
    summary: the comment above PR-Review still says its Stuck is always a Halt, directly above the added on_reroute
    fix: "Reword the two comment lines at 619-620 as the diff rewords the same comment in contracts/recipes/tapestry-recipe.yaml: its Stuck halts the run, except a marked send-back's reroute to Code-Bouncer."
    origin: lens:code-affected-set
  - id: F11
    severity: NIT
    class: consistency
    location: contracts/stencils/loom/loom-fragment-loosened-check.md:2
    summary: the header comment lists every current consumer of the fragment, so a new includer forces an edit here
    fix: "Replace the sentence naming each brief with one that names the group: every brief of a session that commits code includes this fragment by name, with no pipeline."
    origin: handler
---

### [MEDIUM] The pr-findings exemption in resolveRunID is untested

**Location:** internal/loomcli/arm.go:69
**Issue:** `resolveRunID` takes the first positional as a run id unless the verb is excepted. The diff adds `pr-findings` to the exceptions, but `TestResolveRunID` has no row for it. Without the exception the verb would read the findings file as a run id and refuse.

### [LOW] A crash mid send-back leaves two friction notes for one send-back

**Location:** internal/loomcli/prfindings.go:364
**Issue:** The verb stores the file, writes the note, then updates the record. A crash after the note write leaves the record unmarked, so the re-run computes the same send-back number, and `friction.NotePath` hands out the next free name. D6 asks for one note per send-back.

### [LOW] Full input over a changed subject, and the amendment note, are unpinned

**Location:** internal/shedadapters/bouncer_amendment_test.go#TestBouncer_Amendment
**Issue:** The new rows start a full generation over an unchanged subject. After PR-Review fixes land as commits the subject is changed, which reaches the same branch through the changed-subject path. No row pins that, nor that the review input note is none for an amendment record.

### [LOW] Command-mode loosened-check paths beyond exit 0 with an unruled file are unpinned

**Location:** internal/driftengine/verify_test.go#TestDriftVerify
**Issue:** `storeCaseRulings` now merges file rulings and an uncaused halt after a failing compare, and `passCommand` reuses stored file rulings on an exit 0. The one command-mode subtest covers exit 0 with a file not yet ruled only.

### [LOW] Trailer-line extraction has no direct test

**Location:** internal/driftengine/loosened.go#loosenedTrailerLines
**Issue:** The ruling key is a hash over the diff and these lines, so the last-paragraph rule, sorting and duplicate removal decide when a ruling is reused. The tests use messages with one trailer only.

### [LOW] ShedStatus closure and two sendBackEdge error branches are untested

**Location:** internal/loomcli/landingdeps.go#ShedStatus
**Issue:** The landing tests inject a fake status, so the real closure's mapping of a missing status file to an empty status is never run. `TestPRFindingsVerb` covers a missing PR-Review row but not a missing target row or an entry gate that errors.

### [LOW] Contained test duplicates its covering rows

**Location:** internal/landingshed/prreview_wait_test.go#TestPRReview_WaitSendBackHaltedOnADirtyTreeReroutesOnTheNextCallOnACleanTree
**Issue:** The test walks a dirty halt, then a clean reroute, over one row. Each step's assertions, including the mark kept in the record, are made by `TestPRReview_Wait` and `TestPRReview_Call` rows, so it pins nothing they do not.

### [LOW] Two contained tests pin edges no covering test asserts

**Location:** internal/loomrecipe/recipe_test.go#TestRecipe_PRReviewReroutesToCodeBouncer
**Issue:** `TestDriftRows_EntryGate` asserts no `on_reroute`, and `TestArrasRun_GotoIntoRaddleWrite` runs another recipe, so both tests pin what coverage cannot see. The rubric asks for a `//testtiming:keep` mark with the reason.

### [LOW] docs/overview.md leaves the Drift judge and the parent ask stale

**Location:** docs/overview.md:1085
**Issue:** The diff adds the `pr_review_small_max` and `pr-findings` text but leaves the Drift rows described as judging expected changes only, and the ask described as once per pull request. D3 adds a loosened-check ruling and an uncaused Stuck, and D4 clears the ask on a send-back.

### [NIT] Fixture comment still says PR-Review's Stuck is always a Halt

**Location:** internal/loomrecipe/testdata/tapestry-rows-before-motif.yaml:620
**Issue:** Every copy of this comment in the recipes was reworded; the fixture's copy sits directly above the added `on_reroute`.

### [NIT] Fragment header enumerates its consumers

**Location:** contracts/stencils/loom/loom-fragment-loosened-check.md:2
**Issue:** The header names each brief that includes the fragment. A new includer or a rename makes it stale with nothing to flag it.

## Rejected

- lens:code-tests F-T3, a test reading a `judged.json` with no `files` key: the key is optional in the record's JSON tags, so the older record reads as an empty list with no code path to pin.
- lens:code-tests F-T4 and lens:code-correctness F2, an unconfirmed include resolution for the darn, landing fixer, gatefix-darn and weaver briefs: the darn, gatefix and weaver readers go through `ReadForEngine`, and the landing fixer renders through the seat table, which reads with `ReadSet`; every reader resolves the include.
- lens:code-scope reported no findings.

The recurring behaviours the forks checked and found correct are listed in their lens reports beside this file.
