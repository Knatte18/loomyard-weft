---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/loomshed/darnwrite.go#renderPriorWork
    summary: With no recorded outcome, scratch files alone make a first writer spawn be told an earlier spawn was interrupted, though the task's shared .scratch also holds files from the Drift-Expect agent, the operator or the orch
    fix: "In renderPriorWork, stop treating scratch files alone as evidence of an interrupted writer spawn. With found false and no commits beyond the base, return \"\" whatever the scratch directory holds. Claim an interrupted spawn only when commits beyond the base exist, and list the scratch files after the commit list in that case only. Change the TestRenderPriorWork row \"scratch files alone name an interrupted spawn and a long list is cut with a count\" so scratch files alone render \"\". Move its cap check to a row with one commit and priorWorkListLimit+2 files that expects the interrupted note, the commit, 20 file lines and \"- and 2 more\". Update the renderPriorWork doc comment, the DarnDeps.ScratchDir doc and the doc.go prior-work lines to match."
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: contracts/stencils/darn/darn-template-write.md:42
    summary: "Step 2 still says any prior work other than (none) means the branch commits are the writer's own from an earlier spawn, which contradicts the new note that offers that work only as a likelihood (\"may be present\")"
    fix: "In contracts/stencils/darn/darn-template-write.md Step 2, replace the sentence opening \"When the prior work is not `(none)`, the commits already on the task branch are yours\" with two sentences. The first: when the prior work is not `(none)`, it names the work an earlier spawn of this task left on the task branch and in the scratch directory, and you continue from it and fix what it names, never restarting the change. The second: when it says the earlier spawn was interrupted, read the commits and files it lists before you continue, since that spawn recorded no outcome. Keep the `(none)` sentence as it is."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/loomcli/wiring_darn_test.go:892
    summary: No test pins the production wiring, so a wrong range, a wrong base or a wrong scratch path passes every test. The unpinned parts are darnDeps' BranchCommits (CommitLog from taskMergeBase to HEAD), its ScratchDir and fabricengine.ScratchLinkPath, and Darn.BranchCommits is missing from TestWire_DarnRecipe's darnSeams
    fix: "In internal/loomcli/wiring_darn_test.go, add Darn.BranchCommits to the darnSeams list near line 81. Beside TestDarnDeps_LatestOutcomeNamesTheRowsFindingsFile, add a test on the same hermetic pair fixture that makes one code commit with a known subject on the task branch. It then asserts that darnDeps(loc, shedrun.SelfRunID, shedrun.RecipeDarn).BranchCommits() returns exactly one line, the commit's short SHA followed by a space and that subject. It also asserts that .ScratchDir equals filepath.Join(loc.AnchorPath(), \".scratch\"), written as a literal join and not by calling ScratchLinkPath."
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: _lym/landing/summary.md:39
    summary: The Merge danger blast radius names loomshed alone, but the diff also changes loomcli's darn wiring, adds an exported fabricengine function and makes the shedrecipe DarnWrite row refuse an Env without the two new seams
    fix: "In _lym/landing/summary.md, change the blast radius line to name loomshed, loomcli (darnDeps), fabricengine (ScratchLinkPath) and shedrecipe (the DarnWrite row's new required Env.Darn fields), keeping the two-way door."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/loomshed/darnwrite.go#renderPriorWork
    summary: "After a recorded outcome followed by a spawn interrupted before it recorded one, the note's \"That spawn ended stuck.\" and its kept findings read as the latest spawn's ending, and nothing says a later spawn may have been interrupted"
    fix: "In renderPriorWork's found branch, when work.commits or work.scratchFiles is non-empty, add one line after the outcome and findings lines. The line says a later spawn may have been interrupted before it recorded an outcome, so the commits and files listed below can go beyond the spawn that ended so. Add a TestRenderPriorWork row with a recorded Stuck and one commit that expects the line."
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/loomshed/darnwrite.go#listScratchFiles
    summary: An unreadable or vanishing entry anywhere under .scratch now halts the Darn-Write row before its spawn, though the listing feeds only an advisory note. A dangling link and the walk's failure path have no test
    fix: "In listScratchFiles' WalkDir callback, when err is non-nil for a path other than \".\", skip it: return fs.SkipDir for a directory entry and nil otherwise. Keep returning the error for the root \".\". Add TestDarnWrite_Call rows for a dangling scratch link, which expects empty prior work, and for a scratch tree holding a subdirectory made unreadable through permkit.Chmod, which expects the call to go on and list the readable files."
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/loomshed/darnwrite.go#listScratchFiles
    summary: The cap cuts a lexical listing of the whole recursive tree, so one deep directory under .scratch fills the 20 slots and pushes a top-level half-finished file into the count
    fix: "Make listScratchFiles list the direct children of dir only, each subdirectory rendered with a trailing slash, in lexical order. Update its doc comment, the branchWork.scratchFiles doc and the TestDarnWrite_Call interrupted row, whose expected list becomes \"- measure.md\" and \"- runs/\"."
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/darnengine/spec.go:64
    summary: "The DarnInputs.PriorWork field doc still says \"empty on a first spawn\" and that it names what an earlier spawn left unfinished or broken, but it now also carries an interrupted spawn's commits and files"
    fix: "Rewrite the doc comment at internal/darnengine/spec.go:64 to say PriorWork names an earlier spawn's recorded outcome or interruption and the work it left on the task branch, and that it is empty when no earlier spawn left anything."
  - id: F9
    severity: NIT
    class: design
    origin: lens:code-tests
    location: internal/loomshed/darnwrite_test.go#TestRenderPriorWork
    summary: No row pins the scratch-file list following a recorded outcome; only the commit list has a recorded-outcome row
    fix: "Add one scratch file to the work of the TestRenderPriorWork row \"an earlier entry lists the branch's commits after its findings\". Extend its expected text with \"These files survive under the scratch directory /w/.scratch:\" and the file's bullet line after the commit list, before decisionsWinLine."
---

The diff does what the board entry asks.
A writer spawn interrupted before it recorded an outcome is now told about its commits beyond the base and its surviving scratch files, where before it was told `(none)`.
The findings are a false positive in the scratch-only branch, a stencil sentence left stronger than the new note, a production seam no test pins, and smaller edges.
None blocks.

Focus entries, each driven:

- `latestDarnOutcome` against `renderPriorWork`: with no recorded entry the note is now true. With a recorded outcome followed by an interrupted spawn, the commits are listed but the ending is the older spawn's (F5).
- The scratch listing at the first spawn: the Drift-Expect agent, the operator or the orch can leave files there, and they trigger a false interrupted note (F1).
- `listScratchFiles`, `fileExists` and `bulletList`: a dangling link reads as empty, and a nested symlink is listed and never followed. A walk error halts the row (F6), and the lexical recursive cut can drop what matters (F7). A file name with a newline breaks only the list's markup, and that is not raised.
- The `BranchCommits` seam: `rev-list base..HEAD` is newest first. Record-Write, the description commit and the Drift-Expect seal all commit under `_lym`, a link to the records side, so the summary's design claim holds and the first spawn's list is empty. After a parent merge the merge commit itself would be listed under a header that claims only "the task branch holds", which is true. A run with no origin record halts earlier but no longer newly, since the darn verify gate form already requires `VerifyMergeBase` (`internal/shedrecipe/recipe.go:118`).
- The writer stencil: F2.
- Coverage of the wiring: F3; the scratch list after a recorded outcome: F9.
- The change description: its design claim holds; its blast radius is short (F4).

### [MEDIUM] Scratch files alone falsely name an interrupted spawn

**Location:** internal/loomshed/darnwrite.go#renderPriorWork

**Issue:** The task's `.scratch` link is shared by every agent of the task. Drift-Expect runs an agent right before Darn-Write, and its edit directive (`contracts/stencils/drift/drift-template-expect.md:17`) sends scratch files to `.scratch/` with no rule to remove them. With no recorded outcome, any such file makes the first writer spawn read "An earlier spawn of this task was interrupted… continue it instead of starting over". The stencil then tells it never to restart the change. That is the mirror of the bug being fixed.

### [MEDIUM] Writer stencil asserts ownership the new note only offers as likely

**Location:** contracts/stencils/darn/darn-template-write.md:42

**Issue:** Step 2 reads any prior work other than `(none)` as "the commits already on the task branch are yours from an earlier spawn". The new interrupted note says only that work "may be present", and in the scratch-only case it lists no commits at all. The prompt as a whole is no longer consistent with itself.

### [MEDIUM] No test pins darnDeps' BranchCommits and ScratchDir wiring

**Location:** internal/loomcli/wiring_darn_test.go:892

**Issue:** Every new test drives a fake `BranchCommits` and a temporary `ScratchDir`. No test runs `darnDeps(...).BranchCommits()` over a git fixture or asserts its `ScratchDir`. No test references `fabricengine.ScratchLinkPath`, and `TestWire_DarnRecipe`'s `darnSeams` lacks `Darn.BranchCommits`. A reversed range, a wrong base or a path off the link would pass the suite.

### [LOW] Merge danger blast radius names loomshed alone

**Location:** _lym/landing/summary.md:39

**Issue:** The diff also changes `loomcli.darnDeps` and adds the exported `fabricengine.ScratchLinkPath`. It also makes the shedrecipe `DarnWrite` row refuse an Env without the two new seams, so the radius names less than the diff can break.

### [LOW] Note after a recorded outcome hides a later interrupted spawn

**Location:** internal/loomshed/darnwrite.go#renderPriorWork

**Issue:** An interrupted halt appends no history entry (`internal/shedengine/failedstop.go:85-95`), so `latestDarnOutcome` returns the spawn before it. The note then states that spawn's ending as the last one, though the listed commits may come from a later spawn that was killed.

### [LOW] A walk error under .scratch halts the row

**Location:** internal/loomshed/darnwrite.go#listScratchFiles

**Issue:** The `WalkDir` callback returns every error, and `Call` returns it before the spawn. An unreadable subdirectory, or an entry that vanishes during the walk, now stops a row that never read the directory before, though the listing is only advisory.

### [LOW] Cap order can drop what a resumed writer needs

**Location:** internal/loomshed/darnwrite.go#listScratchFiles

**Issue:** The listing is recursive and lexical, and the cut keeps its first 20 entries. One cloned tree or work directory under `.scratch` fills the list and pushes the half-finished top-level file into "- and N more". The whole tree is also walked into memory before the cut.

### [LOW] DarnInputs.PriorWork doc is stale

**Location:** internal/darnengine/spec.go:64

**Issue:** The doc still says the field is empty on a first spawn and names what an earlier spawn left unfinished or broken. It now also carries the interrupted note and the commit and file lists.

### [NIT] Scratch list after a recorded outcome is unpinned

**Location:** internal/loomshed/darnwrite_test.go#TestRenderPriorWork

**Issue:** The recorded-outcome row covers the commit list only. Nothing fails if the scratch list after the outcome lines is dropped or reordered.

## Rejected

- code-scope F1 (lists after a recorded outcome are unlicensed scope): the goal's re-step after an interrupted halt includes a run whose earlier spawn recorded an outcome and whose next spawn was killed. Listing the work after a recorded outcome is what makes that re-step's note true.
- code-affected-set F3's claim that a missing origin record now newly halts the row: the darn verify gate form already requires `VerifyMergeBase` (`internal/shedrecipe/recipe.go:118`), so such a run failed before. The blast-radius part is kept as F4.
- code-affected-set N1 (`ScratchLinkPath` beside `scratchLinkPath`): they take different inputs, the current location against a named slug's worktree, and both build from the one `scratchDirName` constant, so they cannot drift.
- code-tests LOW on `ScratchLinkPath` having no test: folded into F3. The `listScratchFiles` failure-path and dangling-link coverage is folded into F6.
