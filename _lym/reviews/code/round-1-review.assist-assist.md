---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    origin: lens:code-affected-set
    location: internal/reportengine/landings_test.go:500
    summary: The recursive retention test omits the seam declarations required for nested structs.
    fix: Add a //lym:seam directive above TestLandingJoiner_HoldsNoParsedBoard for every struct typePathTo traverses, including Sources, Report, boardHistory and their nested struct types; keep the declarations synchronized with the traversal.
  - id: F2
    severity: BLOCKING
    class: consistency
    origin: lens:code-tests
    location: internal/reportengine/landings_test.go:502
    summary: The retained contained test lacks the required testtiming keep annotation.
    fix: Add //testtiming:keep pins absence of parsed board entries from the joiner type graph above TestLandingJoiner_HoldsNoParsedBoard alongside its seam directives.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/reportengine/batten.go:62
    summary: The fixed lookback drops checkpoints required to join landings inside the report window.
    fix: Separate landing-checkpoint discovery from the bounded status walk or backfill the checkpoint metadata needed by the landing join, including untagged and partial runs; preserve the former join results without retaining parsed historical records, and add an old-checkpoint row to the existing landing test.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/reportengine/boardhistory.go:56
    summary: A slug absent from the boundary board loses its earlier labels and changes landing classification.
    fix: Backfill the last historical labels needed by selected landing slugs that the boundary snapshot and window commits do not supply; retain only compact label data and keep each parsed board transient. Add a landing-table row whose labelled entry was pruned before the boundary snapshot.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/reportengine/boardhistory.go:52
    summary: The oldest included commit's first parent does not necessarily supply the pre-window board context.
    fix: Select the boundary history required by the existing label lookup across reachable commits instead of assuming chronological parent timestamps; preserve lookup order and avoid duplicate reads. Add a backwards-timestamp history to the landing test with the expected historical label stated literally.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/reportengine/runs_test.go:229
    summary: The batten regression test still passes when bounded widening degenerates into a whole-history walk.
    fix: Extend TestCompute_ReadsBattenStatusesOnlyOverEachRunsSpan with literal expected records-query floors and counts, covering an initially bounded read, successful bounded widening, and no-seed termination after no-progress fallback; assert those queries alongside snapshots and file-read counts.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/reportengine/landings_test.go:468
    summary: No regression row pins the board-floor adjustment for a time-join opening before the parent window.
    fix: Convert the existing board-read test to a table and add an owner opening before the normal floor, an early done commit between that opening and the floor, and a later done commit after an in-window parent commit; assert that the parent commit remains unmapped and the extended board interval is read once.
---

### [BLOCKING] Declare the reflective traversal's seams

**Location:** `internal/reportengine/landings_test.go:500`

**Issue:** The test declares only `landingJoiner` and `boardengine.Task`.
Its helper recursively visits struct fields through pointers, slices, arrays and maps.
It also reaches `Sources`, `Report`, `boardHistory` and their nested structs.
`PATTERN-test-speed` requires a seam declaration for every traversed struct, including nested types.
This is an explicit repository constraint, whose required severity overrides the general mechanical-gate exclusion.

### [BLOCKING] Mark the retained test's independent assertion

**Location:** `internal/reportengine/landings_test.go:502`

**Issue:** The supplied redundancy report lists this test as contained by the shared timing-walk test.
The retention assertion checks the absence of a board-entry type in the joiner's reachable fields.
The timing test checks run count and records-walk count; neither assertion checks board retention.
Keeping the test is justified, but `PATTERN-test-speed` requires the missing keep annotation.

### [MEDIUM] Preserve older checkpoints needed by current landings

**Location:** `internal/reportengine/batten.go:62`

**Issue:** With Since at hour 300 and FreshDays 1, the initial records floor is hour 108.
An untagged pair's checkpoint at hour 100 disappears even if it lands at 310 and becomes done at 320.
No archived full run triggers seed lookup for that pair, so the index never widens for it.
`timeJoinOwners` receives no checkpoint and leaves the landing unmapped.
The retro caller also drops the landing because its slug is empty.
The summary acknowledges this exclusion, but the Goal authorizes bounding memory and reads, not dropping valid joins.
The correctness, affected-set and scope lenses independently identified this same defect.

### [MEDIUM] Preserve labels of entries pruned before the boundary

**Location:** `internal/reportengine/boardhistory.go:56`

**Issue:** Suppose a slug has feature labels at hour 1, is removed at hour 5, and the board floor is hour 10.
An exact-joined landing at hour 20 formerly obtained the labels from hour 1.
The new reader retains only hour 5 and subsequent boards, so the landing becomes other with a missing-entry finding.
For a bug label this also removes the landing from freshness classification and fix aggregates.
The old lookup searched past absent entries; retaining only one boundary board does not preserve that contract.
The summary's design note is not a scope amendment.
The correctness, affected-set and scope lenses describe the same defect here.

### [MEDIUM] Do not infer chronological boundaries from parent pointers

**Location:** `internal/reportengine/boardhistory.go:52`

**Issue:** Consider a linear board history A@1 → B@20 → C@5 → D@40 with floor 10.
A holds a bug-labelled entry, B removes it, C holds it with feature labels, and D removes it.
The window query retains B and D; selecting B's parent adds A and omits C.
For an exact landing at hour 35, the old walk finds C's feature label, while the new history supplies A's bug label.
`gitrepo.Log` filters each reachable commit by time and supplies no monotonic-parent guarantee.
This differs from F4 because the retained boundary has the slug but supplies the wrong historical value.

### [MEDIUM] Assert the bounded records walk itself

**Location:** `internal/reportengine/runs_test.go:229`

**Issue:** Snapshot lengths and status-read counts do not distinguish bounded widening from an unbounded fallback.
Deleting the lookback doubling at `batten.go:104` repeats the same window and triggers the whole-history fallback.
Deleting the walked-count assignment at `batten.go:79` also forces that fallback.
Both mutants survived the supplied report, and neither change violates the test's current assertions.
The Goal requires a test pinning this sweep fix, so query floors and termination need assertions.
The tests lens proposed BLOCKING; MEDIUM fits this coverage gap because no listed constraint assigns it BLOCKING.

### [MEDIUM] Exercise the earlier-owner board floor

**Location:** `internal/reportengine/landings_test.go:468`

**Issue:** The new test's normal floor is hour 56 and its checkpoint opens at hour 85.
It never exercises the backward floor adjustment in `readLandings`.
The existing landing table also starts its window before its checkpoints.
An early done mark can close an owner's window before the report window opens.
If the floor adjustment disappears, that mark can be omitted and a later done mark can falsely admit a current landing.

## Review coverage

Reviewed commits from `2dbfdc314b13ae35f459c80406748bc8e6bbfdce` through `2edb8ba9a8ad937e30d61a583714e79b501edd31`.
The Root-Cause trailer names the removed unbounded walks and revision caches.
The diff changes no persisted schema, config meaning or outward write, so the two-way merge-danger claim holds.
Cost reaches the timing changes; raddle reads current store files and archive-tag usage; replies consumes supplied entries.
No additional unbounded revision cache was identified in those latter report paths.
The shared index keeps returned snapshots valid when later selection widens it.
Missing status files preserve earlier readable snapshots, and absent seeds terminate through the whole-history fallback.
Done spans preserve reopened-slug behavior over the retained commits.
The board-history state retains compact label and done spans, not parsed boards.
The batten reader retains statuses for one span lookup and returns the run snapshots existing classifiers consume.
The reflective test catches a concrete board-entry field added to its traversed type graph, not arbitrary interface caches.
The changed tests use in-memory fixtures and fixed timestamps, with no added waits.

## Rejected

- Dropping the duplicate-done bookkeeping at `boardhistory.go:101` does not change the observed join result.
  Its mutation survival alone does not establish a missing behavioral assertion.
- Removing the retention test as redundant would discard an assertion its reported covering test does not make.
  F2 records the required keep annotation instead.

## Verification

The foreground command `timeout 3600s lym gate test --tier 0 ./internal/reportengine` could not acquire a gate slot.
The hub's `_bolt/.lym/gate` and run-process directories were read-only in this session; no tests ran.
The earlier invocation used an invalid tier spelling and ran no tests either.
The supplied reach, mutation and redundancy reports were used as review evidence, not as newly executed checks.
