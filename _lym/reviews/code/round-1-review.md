---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: handler
    location: internal/battencli/startguard.go:96
    summary: A finished motif now carries discussed, but the batten start guard refuses a hand start only for draft, so lym batten run starts it with no release while the README says nothing starts it.
    fix: "In internal/battencli/startguard.go, make the Draft probe at line 96 true for a board status of boardengine.StatusDraft or boardengine.StatusDiscussed, and word the refusal at line 170 to name the status the entry carries (a draft task or a discussed task), keeping the release command as the way forward. Update the Draft field comment at line 43 and the refuse comment at line 117 to name discussed beside draft. Add a startguard test row for a discussed entry with no status file that expects the refusal naming the release command."
  - id: F2
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/boardengine/board.go:196
    summary: The new SetRunStatus skip for an entry that already carries the status has no test; TestSetRunStatus would still pass with the clause removed.
    fix: "In internal/boardengine/board_test.go TestSetRunStatus, add a row that seeds an entry whose status is already 'running · Webster', calls SetRunStatus with that same status, and asserts the write was skipped, for example by seeding with a distinguishing Revised value or reading the board file bytes before and after and requiring them equal. The test runs with SkipGit, so assert on the board file, not on a Bolt commit."
  - id: F3
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/battencli/commitstatus.go:138
    summary: Batten's board write has no test for a failing SetBoardStatus (commit and spawn must still run) and none for the real SetBoardStatus closure that maps the run id to the task slug.
    fix: "In internal/battencli/commitstatus_test.go, add a row to TestNewCommitStatusSeam_WritesBoardStatusBeforeRunShed (or a sibling subtest) whose SetBoardStatus stub returns an error at Worktree-Create and assert the seam returns nil and Commit and Spawn were still called. For the real closure in battenCommitStatusDeps, add a hub-fixture test (hubforge) that seeds a board entry, drives the closure, and asserts the entry's status reads 'running · <row>'; the loom analogue is TestWiredCommitStatus_WritesFinishedStatus."
  - id: F4
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/shedengine/run.go:299
    summary: The start write has no test for a first row that is entry-gated, where started is recomputed after the gate re-read.
    fix: "In internal/shedengine/run_commitstatus_test.go next to TestStep_StartWriteNamesTheFirstRow, add a case with an empty-current_producer seed whose first row has an EntryGate that holds, and one whose gate skips to the second row. Assert for the holding gate that the status during the first row's call names that row and CommitStatus saw it once, and for the skipping gate that the second row's call sees its own name and CommitStatus is not told the first row twice."
  - id: F5
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/boardcli/text.go:54
    summary: The decision record names the --text listing and the list layer as places a finished motif must read as finished, and no boardcli test asserts discussed in either.
    fix: "In internal/boardcli/verbs_test.go (the list steps), seed a task with status 'discussed' and assert the JSON list row carries layer 'Discussed' and the --text listing line ends with '[discussed]'."
  - id: F6
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/boardengine/layer_test.go:58
    summary: The Discussed layer and render tests leave two stated behaviours unpinned, that a draft entry carrying the discussed flag stays Draft and that the Discussed table carries the Recipe and After columns.
    fix: "In internal/boardengine/layer_test.go TestComputeLayers add a row with Status draft and Discussed true expecting layer Draft. In render_test.go TestRenderDiscussedSection assert the Discussed table's header row carries the Recipe and After columns, as doc.go:271-275 states."
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/boardengine/doc.go:244
    summary: The list of statuses that are not run statuses and the set-status and merge help text name draft only, and do not name discussed, which a null status also releases.
    fix: "In internal/boardengine/doc.go:244 add 'discussed' to the list of statuses that are not run statuses. In internal/boardcli/cli.go set-status help (the Long text near line 640, the status field line, and the merge help near line 879-881) say that null releases a draft or discussed task."
---

### [MEDIUM] Hand start of a discussed entry is not refused

**Location:** internal/battencli/startguard.go:96
**Issue:** Before the diff a finished motif was left at `draft`, so `lym batten run <slug>` on it was refused with the release command named. The finished motif now carries `discussed`, which the guard's Draft probe (`== boardengine.StatusDraft`) does not match, so a hand start goes through and claims the entry again. Dobby does not start it, but the README section the diff adds says "nothing starts it until its status is cleared", and `decision.go`, `boardengine/doc.go` and the board skill say the same. The code and those claims disagree on the hand path. Two lenses reached this independently.

### [MEDIUM] SetRunStatus equal-status skip has no test

**Location:** internal/boardengine/board.go:196
**Issue:** The diff adds `task.Status != nil && *task.Status == status` to `SetRunStatus` so a repeated transition makes no Bolt commit, and the batten seam and the start write both lean on it. `TestSetRunStatus` was not changed and its rows (no status, done, absent slug) pass with the clause deleted.

### [LOW] Batten board-write failure path and real closure untested

**Location:** internal/battencli/commitstatus.go:138
**Issue:** The header says a failed board write only warns and the commit goes on, but no row makes the stub fail. The real `SetBoardStatus` closure, which relies on a batten run id being the task slug, runs in no test.

### [LOW] Start write untested for a gated first row

**Location:** internal/shedengine/run.go:299
**Issue:** `started` is recomputed after the entry-gate re-read. The new test seeds an ungated first row, so a change to that recomputation would pass.

### [LOW] Discussed not asserted in the list layer or --text listing

**Location:** internal/boardcli/text.go:54
**Issue:** The record asks that a finished motif show as finished in the list and the `--text` listing. The code is generic over status and the layer is pinned in `boardengine`, but no `boardcli` test asserts either output for a discussed entry.

### [LOW] Two Discussed behaviours unpinned

**Location:** internal/boardengine/layer_test.go:58
**Issue:** The new rows set status and flag together, so a regression to a flag-keyed layer would pass, and the Recipe and After columns that `doc.go` states for the Discussed table are not asserted.

### [LOW] Status docs and help omit discussed

**Location:** internal/boardengine/doc.go:244
**Issue:** The list of non-run statuses and the `set-status` and merge help describe release in terms of draft alone.

## Rejected

- Existing hubs change behaviour silently (lens:code-correctness): the template is seed-only by design, `lym config reconcile` reports each missing motif block under `lacking`, a profile without a motif block is refused with a way forward, and the change description lists the hand step. No config value changes meaning, so `PATTERN-config-reinterpretation` does not apply.
- `closedStatus` comment in adoptcli/runreport.go says "such as the draft a cancel writes" (lens:code-affected-set): the behaviour is right for `discussed` and "such as" does not exclude it.
- Start write resume case needs a test (lens:code-tests finding 8): the lens itself found no gap, and the running-to-running path is unchanged.
- `lym loom status` has no loomcli-level test (lens:code-tests item 10): the diff does not change `internal/loomcli` status code; the status file is the source and `TestStep_StartWriteNamesTheFirstRow` pins the producer in the file during the first call, so this is not a gap.
- Empty recipe name in `prepareRunSettings` (lens:code-correctness): the recipe comes from the seed's resolved recipe, so an empty name does not reach it.

Focus notes. Issue #58: no path resolves motif's profile through tapestry; `ProfileRecipe` and `profile_recipe` appear only under `docs/research`, and `migrate_test.go` expects the motif blocks. Issue #49: the diff leaves `internal/loomcli` status reading unchanged; `lym loom status` reads the status file, which the start write now fills with the first row before its call. The change description's claims match the diff apart from the hand-start point in F1.
