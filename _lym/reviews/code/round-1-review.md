---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/boardengine/raddlefile_test.go
    summary: The one-task outcome's re-point of dependents onto the single filed task has no test, though the D10 criterion names it.
    fix: In the TestFileRaddle table of internal/boardengine/raddlefile_test.go add a tasks-shape FileRaddle case. Put one open entry that depends on the run's entry in it and assert its depends_on now names the one filed task. Add a done dependent on the run's entry and assert it stays unchanged.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/loomshed/raddlefile.go:209
    summary: A repaired family filing in the AlreadyFiled case can claim a run status back onto the converted project, because stageAndFile tests the entry read before repairFamily cleared its status.
    fix: In Call, pass repaired (or resumed) to stageAndFile, or clear entry.StatusProducer after repairFamily succeeds, so ClaimStatus is never called for a filing found as its own. Add a loomshed row where the repaired entry's StatusProducer is another producer and assert claimed stays empty.
  - id: F3
    severity: LOW
    class: design
    origin: handler
    location: internal/boardengine/raddlefile.go:194
    summary: endsAsFamilyProject compares the entry to arrasProjectOf only through scopeChanged, which does not read profile, run_settings or notify_on_land, so the lock admits a project that keeps them although D6 lists them as cleared.
    fix: In endsAsFamilyProject also require now.Profile to be empty, now.RunSettings to be nil or empty and now.NotifyOnLand to be false (the fields withoutRunFields clears that scopeChanged ignores). Add TestCheckRunLocks_FamilyProject rows that leave each of the three set and expect a refusal.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomcli/wiring.go:250
    summary: Only the skipAfterFamilyFiling wrapper is tested. Nothing pins that commitStatusDeps applies it for a Raddle-File recipe and only then, or the real Entry, ClaimStatus, MarkFiled and ClearStatus wiring in raddlewiring.go.
    fix: Add a test in internal/loomcli that builds commitStatusDeps for the arras recipe, writes the filing mark in the prime and asserts SetBoardStatus writes nothing. Assert the same call still writes for a recipe with no RaddleFile row.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/boardengine/raddlefile_test.go
    summary: The D8 criterion that a null status written by upsert, merge or batten cancel's claim clear releases no member is exercised only by SetStatus(plan, nil) on a status that is already nil.
    fix: In the family subtest set a run status on the converted project first so the nil write is a real change. Cover the upsert and merge writes there too, or name the existing test that already covers them.
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/boardengine/runlock.go:171
    summary: Several new branches have no row. They are the lock rows for an order or body that differs from the after sequence, a non-draft part, a part already loaded and no added parts. The others are the validateWrite refusal in repointDependents, the ClearProjectRunStatus no-ops, the no-prior-record restore branch, and MarkFiled or ClearStatus failing in repairFamily.
    fix: Add the missing rows to TestCheckRunLocks_FamilyProject and the TestFileRaddle family subtests in internal/boardengine. Add f.failing rows for mark_filed and clear_status in the TestRaddleFile table of internal/loomshed/raddlefile_test.go. Fold them into existing tables and subtests, no new tests.
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/loomshed/raddlefile.go:177
    summary: family is false whenever the proposal fails to parse, so a done store with an unparsable proposal runs finish with family false and EndEntry would mark the converted project done.
    fix: In Call, when filed or resumed holds and parseErr is not nil, return p.fail("parse the proposal", parseErr) before finish, so EndEntry is never reached for a filing found as its own.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: docs/overview.md:1302
    summary: The line says a raddle run ends the entry done in its own Raddle-File, which is no longer true for an arras family filing that turns the entry into the family's project.
    fix: In docs/overview.md line 1302 add the family exception. Say the arras family filing turns the entry into the family's project and that no status write of the run reaches it afterwards.
  - id: F9
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: contracts/recipes/raddle-recipe.yaml:4
    summary: The comment says Raddle-File ends the run's own entry, but the row is shared with arras, whose family filing does not end it.
    fix: Reword the comment at contracts/recipes/raddle-recipe.yaml line 4 to say it ends the run's own entry unless a family filing turned it into the family's project.
---

### [MEDIUM] One-task re-point of dependents has no test

**Location:** internal/boardengine/raddlefile_test.go
**Issue:** The D10 criterion says an entry depending on an arras entry depends, after the filing, on the filed task or on the family's last parts.
Only the family subtest has a dependent on the run's entry.
No test files a non-family shape with a dependent on it, and none has a done dependent that must be skipped.
The code in `repointDependents` is correct on the cases I ran, so the gap is coverage, not a defect.

### [LOW] Repaired filing can claim a status back onto the converted project

**Location:** internal/loomshed/raddlefile.go:209
**Issue:** `Call` reads the run's entry once, before `repairFamily` clears its run status.
When the filing is found as its own through `AlreadyFiled`, `stageAndFile` runs in store-only mode but still tests the stale `StatusProducer`.
If that producer is another one, `ClaimStatus` writes a run status onto the converted project after the clear.
The project then ends with a run status, against the acceptance criterion that it has none after the resume.
It needs a stale foreign status on the entry when the filing is repaired.

### [LOW] Lock admits a project that keeps profile, run settings or notify-on-land

**Location:** internal/boardengine/raddlefile.go:194
**Issue:** D6 lists `profile`, `run_settings` and `notify_on_land` among the fields the admitted project has cleared.
`scopeChanged`, which `endsAsFamilyProject` leans on for the derived-shape comparison, does not read those three.
A write that converts the entry but leaves them set therefore passes the lock.
`endsAsSketch` has the same shape, but D6 binds this admission to the derived project exactly.

### [LOW] Wiring of the skip and the Raddle-File seams is untested

**Location:** internal/loomcli/wiring.go:250
**Issue:** `TestSkipAfterFamilyFiling` pins the wrapper only.
The `recipeHasEngine` condition, the prime-side mark read under `ResolveRunID`, and the real `Entry`, `ClaimStatus`, `MarkFiled` and `ClearStatus` seams have no test.
The wiring was traced by reading and is consistent, so this is a coverage gap.

### [LOW] D8 null-status criterion exercised on an already-nil status

**Location:** internal/boardengine/raddlefile_test.go
**Issue:** The family subtest writes `SetStatus("plan", nil)` after `ClearProjectRunStatus` has already left the status nil, so the write changes nothing.
The criterion also names upsert, merge and batten cancel's claim clear, which have no row in this diff.

### [LOW] Untested branches of the new code

**Location:** internal/boardengine/runlock.go:171
**Issue:** The lock table lacks rows for `order` read from `after` and not from the entry, a changed body, a non-draft part, a part already loaded and no added parts.
`repointDependents` has no row for the `validateWrite` refusal, and `ClearProjectRunStatus` none for its no-op branches.
`convertRunEntry` has no row for an entry with no former record, and `repairFamily` none for a failing `MarkFiled` or `ClearStatus`.
Each is a branch the diff adds with no row that would fail if it broke.

### [LOW] Unparsable proposal with a done store ends the converted project

**Location:** internal/loomshed/raddlefile.go:177
**Issue:** `family` requires `parseErr == nil`.
With a done store and a proposal that no longer parses, `family` is false, so `finish` calls `EndEntry` and marks the converted project done.
It needs a corrupt proposal file next to a done store, which is unlikely.

### [LOW] Overview still says a raddle run ends its entry done

**Location:** docs/overview.md:1302
**Issue:** The sentence "A raddle run ends the entry done in its own Raddle-File, never as a Sketch" no longer holds for an arras family filing.
The neighbouring `internal/loomcli/doc.go` text was updated, so the two now disagree.

### [NIT] Raddle recipe comment says Raddle-File ends the entry

**Location:** contracts/recipes/raddle-recipe.yaml:4
**Issue:** The comment on the Raddle-File row says it ends the run's own entry.
The row is shared with the arras recipe, whose family filing does not end it.

## Rejected

- Code-tests, no test connects the loomshed repair rows to the board's `HoldsRaddleFiling`: each layer is pinned against its own seam, and a joined test would duplicate both.
- Code-tests, `DoneArrasRunIsRemovedWithItsFilingMark` may not assert the mark is gone: `removeRunRecords` removes the whole run directory, so the mark's removal follows from the row it already asserts.
- Code-tests, the family subtest does not assert `isolated`, `profile`, `run_settings`, `pull_request` or `notify_on_land` cleared: kept only as the lock gap in the third finding, not as a separate test finding.
- Declared PATTERN changes: the constraints section declares none, and no commit carries a `Pattern-Departure:` trailer, so nothing is raised.
- Board entry: `lym board get` returned an invalid-JSON error, so the entry was not read. The decision record's `## Goal` was used as the measuring stick.
