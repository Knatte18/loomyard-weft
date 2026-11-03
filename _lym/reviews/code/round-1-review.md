---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/loomcli/discussionrecord_integration_test.go:101
    origin: lens:code-tests
    summary: seedDiscussedRecord has no row for a record or support log already on disk, a flagged entry whose record file is missing, or a returned board or write failure
    fix: In TestSeedDiscussedRecord_Real add rows to the same table. (1) A discussed entry whose pair already holds a different decision-record.md and a support-log.md; assert the record is overwritten with the stored one, the existing support log is kept byte for byte, and both are committed. (2) A flagged entry whose record file was deleted from the board directory; assert seedDiscussedRecord returns false and nil and writes nothing. For the failure path, make the discussion directory unwritable (a regular file at record.dir) in one row and assert seedDiscussedRecord returns false and a non-nil error naming the directory.
  - id: F2
    severity: LOW
    class: design
    location: internal/loomcli/wiring.go:307
    origin: lens:code-tests
    summary: Neither the c.commitStatusDeps call sites at wiring.go:307 and :326 nor the SeedDiscussedRecord line at wiring.go:882 is pinned, so a revert of either passes every test
    fix: Add one test in internal/loomcli that builds the loom wiring for the motif recipe and asserts the commit-status seam it installs writes the draft status when the run finishes (commitStatusDeps for a loomCLI whose recipe is motif returns FinishedStatus draft, and one for tapestry returns empty). Add one assertion in the same area that the Env wire builds for tapestry has a non-nil SeedDiscussedRecord, so dropping wiring.go:882 fails a test.
  - id: F3
    severity: LOW
    class: consistency
    location: docs/research/weaving-glossary.md:24
    origin: lens:code-affected-set
    summary: The glossary still lists the recipe names without motif at lines 24 and 37 and still calls motif reserved for block use only at line 145, though motif is now a runnable task recipe
    fix: In docs/research/weaving-glossary.md add motif to the recipe-name list at line 24 and to shed's recipe list at line 37. Rewrite the motif row at line 145 to say motif is a recipe a board task may name, which runs Discussion alone, and which tapestry runs inline as its Motif block, in place of the sentence that calls it reserved by the shed-sub-run-block task.
  - id: F4
    severity: LOW
    class: design
    location: internal/battenshed/doc.go:157
    origin: lens:code-affected-set
    summary: No code or test shows batten's Land, Worktree-Teardown and the landing notice handling a motif child that ends with no Finalize, though the goal needs the entry to leave the run lock and not restart
    fix: Read the Land and Worktree-Teardown paths in internal/battenshed and internal/battencli for a child run that finished with no Finalize row. Add a test in internal/battenshed that drives a child run whose recipe has no Finalize row to its end and asserts Land and teardown complete with nothing to land and leave the board entry's draft status untouched. If a path assumes a Finalize landing commit, make it treat a Finalize-less child as landing nothing.
---

### [MEDIUM] seedDiscussedRecord failure and existing-file paths are untested

**Location:** internal/loomcli/discussionrecord_integration_test.go:101
**Issue:** `TestSeedDiscussedRecord_Real` has three rows: a discussed entry, an undiscussed one and an absent one, each into a fresh pair. The focus names the failure paths, a flagged entry with a missing record, a record already on disk, and board read errors.
`seedDiscussedRecord` has branches for a support log that already exists, for the overwrite of a record already present, and for returning each failed step; no row reaches them.
The overwrite is the case the landing summary asks to be checked by hand.

### [LOW] Wiring of the draft status and of the seed seam is unpinned

**Location:** internal/loomcli/wiring.go:307
**Issue:** The new tests call `newCommitStatusSeam`, `finishedBoardStatus` and the gate directly. Replacing `c.commitStatusDeps(location)` by `loomCommitStatusDeps(location, c.runID)` at either call site, or deleting the `SeedDiscussedRecord` line at wiring.go:882, fails no test.
The first would leave a finished motif run in the run lock and let dobby restart it; the second would fail the tapestry build at the nil-seam refusal only where a test builds tapestry through the real wiring.

### [LOW] The glossary was left half updated

**Location:** docs/research/weaving-glossary.md:24
**Issue:** The diff edits the glossary's arras row at line 127 and the cartoon line at line 28, but lines 24 and 37 still list the recipe names without motif, and line 145 still says motif is reserved as a recipe name to let a block run a sub-recipe. Motif is now in `shedrun.RecipeNames` and the shed arming table.

### [LOW] A Finalize-less motif child has no batten handling or test

**Location:** internal/battenshed/doc.go:157
**Issue:** A motif run ends with no Finalize and no landing commit, and writes draft on its entry. My search found no code or test in battenshed, battencli or dobbyengine that handles Land, Worktree-Teardown or the `notify_on_land` notice for such a child, and the landing summary lists this under "To check by hand".
The goal asks that a motif run land no code and leave the run lock without a restart; the status write is covered, the consumers of the end state are not.

## Rejected

- Fork code-affected-set finding 2, `Env.SeedDiscussedRecord` against `PATTERN-shed-recipe-registry`: the entry's "never carried as a field of its own" rule governs a row's looked-up setting value, and the "Command row" rule governs a side-action row added to a recipe. The seam is a closure like `ApproveDiscussion` and `CarryOver`, and the block gate is a row of the `BlockGates` table, which the entry names as the home of a block's gate check. The goal asks for the gate to read the stored record into `_lym/discussion`, so no entry text forbids it.
- Fork code-affected-set finding 4, the leftover `arras` in `internal/shedrun/seed_test.go:174` and `docs/research/weaving-glossary.md:127`: the test row pins that the removed name is refused, and the glossary row records that the name is free. Neither names a recipe or a place that uses it.
- Fork code-tests finding on the half-present boundary of the record-only contract: the fork did not establish that the existing `TestBuild_LoomContract` refusal rows leave `recordOnly` unreached, and `validLoom` mutated to drop one of the two rows is the form those rows use.
- Fork code-tests remark that the `Motif` row under the discussion phase in `timingspans.go` has no test: the fork read only the diff stat, and `timing_test.go` is edited in this diff. Not established as a gap.
- Fork code-tests item 7, the `Env.SeedDiscussedRecord` wire line: merged into F2.
- Fork code-correctness: no findings. The cases it ran for the gate, the seed seam, the finished status, the record-only contract, `Expand`, the arbiter description and the row names are correct as written; I found nothing to add.
- Holistic checks that found no defect: the tapestry equality pin compares full expanded rows, config included, and the top-level keys against the testdata snapshot; no `Cartoon` remains in the tree; `Expand` ignoring a block recipe's top-level keys leaves tapestry's keys as before; no commit carries a `Pattern-Departure:` trailer and the directive declares no PATTERN change; the summary's claims about the recipe, the gate, the contract, the draft status and the removal match the diff.
