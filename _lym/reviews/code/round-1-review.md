---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: internal/reportengine/landings_test.go:502
    summary: "TestLandingJoiner_HoldsNoParsedBoard reflects recursively through every struct landingJoiner reaches but carries //lym:seam lines for only landingJoiner and boardengine.Task, against PATTERN-test-speed's one line per walked struct, nested structs included."
    fix: "In the comment block above func TestLandingJoiner_HoldsNoParsedBoard in internal/reportengine/landings_test.go, add one //lym:seam <unit>#<Type> line for each in-module struct typePathTo walks from landingJoiner: at least internal/reportengine#Sources, #Options, #Report, #RunRow, #Landing, #Finding, #PartialRun, #landingInputs, #selectedTag, #landingClaim, #boardHistory, #commitSpan, #labelSpan, internal/gitrepo#LogCommit, internal/gitrepo#TagRef, internal/shedengine#Status, internal/shedengine#HistoryEntry, and every further struct those types hold (list them by walking the field types). Alternatively, narrow typePathTo so it walks only the joiner's own field types and boardHistory, and mark exactly the structs it then walks."
  - id: F2
    severity: BLOCKING
    class: design
    origin: assist:assist
    location: internal/reportengine/landings_test.go:502
    summary: "The redundancy report lists TestLandingJoiner_HoldsNoParsedBoard as contained by TestComputeTiming_WalksTheRecordsBranchOnce; its assertion, that no type the joiner reaches holds a boardengine.Task, is one the covering test never makes, so it is kept, and PATTERN-test-speed requires a kept candidate to carry //testtiming:keep."
    fix: "Add the line //testtiming:keep pins that no type the landing joiner reaches holds a parsed board entry, directly above func TestLandingJoiner_HoldsNoParsedBoard in internal/reportengine/landings_test.go, stacked with its //lym:seam lines."
  - id: F3
    severity: MEDIUM
    class: design
    origin: handler
    location: internal/reportengine/batten.go:228
    summary: "Any run in the window whose slug has no earlier tag and no fresh seed, such as a run with no batten status, makes snapshotsOfRun widen until widen falls back to a walk of the whole records history; that brings back the unwindowed walk, and the checkpoints of every slug it collects can drop the board read's floor to an untagged slug's oldest checkpoint, so the board is read at every commit again."
    fix: "Bound the widening in snapshotsOfRun (internal/reportengine/batten.go) for a run with no earlier tag by a lower limit the run itself supplies, such as its seed's startedAt (the RunRow.startedAt selectRuns and selectInFlight already read) less a margin. Pass that bound in, stop widening once the floor is at or before it, and return no snapshots when no seed lies after it. Add a row to TestCompute_ReadsBattenStatusesOnlyOverEachRunsSpan for an in-window run with no batten status, asserting from w.repo.logQueries that no records walk has a zero Since."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/reportengine/boardhistory.go:43
    summary: "readBoardHistory's board.Log with Since but without FirstParent still traverses every commit of the board branch, since gitrepo.Log's full walk only skips commits before Since and never stops; the records walk at batten.go:74 does the same on every walk and widen, so neither walk is bounded by the window the Goal asks for."
    fix: "Make the walks stop at the floor. For the board walk in readBoardHistory, use gitrepo.LogQuery{FirstParent: true, Since: floor}, which stops at the first older commit, if the board branch is written linearly; otherwise, give gitrepo.LogQuery an option that ends the committer-time walk at the first commit older than Since, and use it in readBoardHistory and battenIndex.walk. Keep the fake Log in fake_test.go matching the chosen semantics."
  - id: F5
    severity: MEDIUM
    class: decision
    origin: lens:code-scope
    location: internal/reportengine/batten.go:91
    summary: "The diff changes what the landing joins produce, which the Goal does not ask for: a checkpoint older than the records walk opens no time-join window, whether it does depends on whether another run widened the walk, and labelsAt drops labels an older board commit held. The Goal asks only that the reads be bounded, and summary.md's Design decisions are the writer's own calls."
    fix: "Either restore the old join results within the bound, or keep the code and defer the departure to the run's parent to record or refuse with lym loom decision add. Restoring means: collect the landing checkpoints from a walk that does not depend on run seeds (for example checkpoint-subject commits back to the oldest selected or untagged slug's pair opening), so widening for another run no longer changes them; and, in readBoardHistory (internal/reportengine/boardhistory.go), keep reading older board commits for each claimed slug the floor's board lacks until its entry is found or the history ends, keeping only its labels."
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/reportengine/runs_test.go:229
    summary: "No test tells a doubled records walk from a jump to the whole-history walk: the mutation survivors at batten.go:79 (b.walked) and batten.go:104 (b.lookback *= 2) both fall through to the full walk, and TestCompute_ReadsBattenStatusesOnlyOverEachRunsSpan's second row still passes."
    fix: "In the second row of TestCompute_ReadsBattenStatusesOnlyOverEachRunsSpan (internal/reportengine/runs_test.go), also assert the Since of every records walk, the w.repo.logQueries entries with a non-empty PathPrefix, as the literal list at(82), at(-86), with no zero-Since walk. Have setup return the expected list beside the snapshots and reads."
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/reportengine/landings_test.go:468
    summary: "No test opens a time-join window before landingWalkStart, so dropping the floor-lowering loop in readLandings (landings.go:149-154) goes unseen."
    fix: "Make TestCompute_ReadsEachBoardCommitOnceFromTheWindowOn table-driven and add a row whose checkpoint lies between the records walk's floor and landingWalkStart: for example Since at(200), FreshDays 1, checkpoint at 160, a board commit marking the slug done at 170, a parent commit at 165 and a later one at 210. Assert that the parent commit at 165 is time-joined to the slug and that the board commits from the one in effect at 160 onward are each read once."
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/reportengine/boardhistory.go:46
    summary: "No test covers readBoardHistory with a non-zero floor and no board commit after it, where the board in effect is the tip (before = tip); breaking it would drop every landing's labels unseen."
    fix: "Add a row to the table F7 makes of TestCompute_ReadsEachBoardCommitOnceFromTheWindowOn, or to TestCompute_Landings, whose newest board commit, labelling the slug feature, lies before the floor. Assert that the landing's kind is feature and that the tip is the only board commit read."
  - id: F9
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/reportengine/batten.go:228
    summary: "No test covers the widening stop for a reused slug's run with no seed after its earlier tag under a non-zero window (earlier != nil && !b.floor.After(*earlier)); removing it would widen to the whole history with the same output."
    fix: "Add a row to TestCompute_ReadsBattenStatusesOnlyOverEachRunsSpan whose slug has an earlier tag before the first floor and a later in-window tag with no batten status after the earlier one. Assert no snapshots, and from w.repo.logQueries that the records walks stop at the first floor at or before the earlier tag, with no zero-Since walk."
  - id: F10
    severity: LOW
    class: design
    origin: assist:assist
    location: internal/reportengine/boardhistory.go:52
    summary: "readBoardHistory takes the board in effect at the floor as the first parent of the oldest commit the time-filtered walk returns; when board commits from different machines carry skewed committer times, a commit older by ancestry but newer than the floor is skipped, and labelsAt can return an older entry's labels than the old newest-first walk did."
    fix: "In readBoardHistory (internal/reportengine/boardhistory.go), take the board in effect at the floor from the first-parent chain: walk first parents from the tip and stop at the first commit older than the floor, which is the board in effect, instead of relying on committer-time order. This combines with F4's first-parent walk."
  - id: F11
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/reportengine/doc.go:46
    summary: "The package doc says the records walk opens a week before the report's window, but Compute opens it a week before landingWalkStart, which is Options.FreshDays before Since."
    fix: "In internal/reportengine/doc.go, reword the sentence to say the first walk opens a week before the start of the parent-branch walk, which lies FreshDays before the window for the convergence report and at the window for the timing report."
---

### [BLOCKING] The reflective joiner test marks two of the many structs it walks

**Location:** `internal/reportengine/landings_test.go:502`

**Issue:** `typePathTo` recurses through pointers, slices, arrays, maps and struct fields from `landingJoiner`.
That reaches `Sources`, `Report` with its `RunRow`, `Landing`, `Finding` and input types, `boardHistory` and its spans, `gitrepo.LogCommit` and the shedengine status types.
`pattern/PATTERN-test-speed.md` ("Reflective tests carry `//lym:seam`") requires one marker per struct the reflection walks, nested structs included.
`lym loom validate-plan` reads these markers for `seam-test-unlisted`, so a later change that adds a board field to, say, `boardHistory` would not list this test.
The repository's precedents mark nested structs, as `internal/loomengine/config_test.go:1030-1031` does.

### [BLOCKING] The kept joiner test lacks its keep marker

**Location:** `internal/reportengine/landings_test.go:502`

**Issue:** The redundancy report lists the test as contained by `TestComputeTiming_WalksTheRecordsBranchOnce`.
That test asserts a run count and a records-walk count; it never checks which types the joiner holds.
The joiner test pins the Goal's "hold no board beyond the one being read", which no read count can see, because a cache would still read each commit once.
So it is kept, and PATTERN-test-speed's test-economy section requires a kept candidate to carry `//testtiming:keep <reason>`.

### [MEDIUM] A run with no seed widens the records walk to the whole history

**Location:** `internal/reportengine/batten.go:228`

**Issue:** When `touching[slug]` holds no fresh seed and `earlier` is nil, the loop's break condition never holds.
`widen` doubles the lookback while each walk finds more commits, then walks the whole history, and only then returns false.
Every first run of a slug lacking a batten status triggers this; `TestCompute_SelectsRuns` treats an absent batten source as a normal case.
After the fallback, `b.checkpoints` holds every slug's checkpoints.
`timeJoinOwners` gives an untagged slug a nil floor, so its newest checkpoint of any age opens a window, and `readLandings` lowers the board floor to that opening.
The convergence report then reads `board.json` at nearly every board commit again, which is the work the Goal set out to remove.

### [MEDIUM] Both history walks still traverse the whole branch

**Location:** `internal/reportengine/boardhistory.go:43`

**Issue:** `gitrepo.Log`'s full walk (`internal/gitrepo/historyreads.go`) runs `commitIter.ForEach` over every reachable commit and returns nil for one older than `Since`; it never stops.
The board walk therefore still decodes about 26 000 commit objects per report.
The records walk does the same on each walk, and each `widen` adds another whole traversal.
The parsed boards are no longer held, so the memory growth is fixed.
The walk itself is not bounded by the window, though, and the fake `Log` skips the same way, so no test sees it.

### [MEDIUM] The bound changes what the landing joins produce

**Location:** `internal/reportengine/batten.go:91`

**Issue:** The Goal asks the joins to keep "only what the joins need", which bounds the reads and keeps the results.
Three results change:
- A landing checkpoint older than the records walk's floor no longer opens a time-join window, so a pair that waited at landing for more than a week before the window comes out unmapped.
- `selectRuns` copies `batten.checkpoints` after every `snapshotsOfRun` call, so whether such a checkpoint counts depends on whether another run's seed widened the walk.
  A partial run or untagged pair never widens the walk itself.
- `labelsAt` returns no labels and files a "no board commit holds an entry" finding for a slug that the board in effect at the floor lacks but an older board held; `labelsOf` found those labels.

`_lym/landing/summary.md` lists the first and third under "Design decisions", which is the writer's own call; no `Added after Discussion` entry records either.

### [MEDIUM] The doubling of the records walk is unpinned

**Location:** `internal/reportengine/runs_test.go:229`

**Issue:** Dropping `b.walked = len(commits)` (`batten.go:79`) or `b.lookback *= 2` (`batten.go:104`) makes the first `widen` fall straight to the whole-history walk.
The second row still finds the seed and reads `l0` and `l270` once each, so both mutants survive.
`widen`'s doc comment states the doubling as its contract, and the Goal asks for each sweep fix to be pinned by a test.

### [MEDIUM] The board floor's move to an older time-join opening is unpinned

**Location:** `internal/reportengine/landings_test.go:468`

**Issue:** The new test's floor is `at(56)` and its checkpoint opens at `at(85)`, after the floor.
Every `TestCompute_Landings` row runs with `Since: at(-500)`, earlier than any of its checkpoints.
No row has a window opening before `landingWalkStart`.
Without the loop, a done mark between such an opening and the floor would be missed, and a later done mark could map the wrong commit.

### [LOW] The tip as the board in effect is untested

**Location:** `internal/reportengine/boardhistory.go:46`

**Issue:** On a quiet board no commit lies at or after the floor, and `readBoardHistory` reads the tip as the board in effect.
Every windowed test has board commits after its floor, so this branch has no row.

### [LOW] The widening stop at an earlier tag is untested

**Location:** `internal/reportengine/batten.go:228`

**Issue:** Row 1 of `TestCompute_ReadsBattenStatusesOnlyOverEachRunsSpan` finds its seed with the floor after `earlier`.
The "never seeded" row of `TestCompute_SelectsRuns` runs with a zero `Since`.
No row exercises the `earlier != nil && !b.floor.After(*earlier)` stop, and removing it would change only how many walks run.

### [LOW] The floor's board in effect assumes committer times follow ancestry

**Location:** `internal/reportengine/boardhistory.go:52`

**Issue:** Take the linear board history A@1 → B@20 → C@5 → D@40 with floor 10.
The time-filtered walk returns D and B, and B's first parent A becomes the board in effect, while C is never read.
For a landing at hour 35, `labelsOf`'s newest-first walk took C's labels; the new history gives A's.
Board writes come from several machines, so skewed committer times are possible, though rare.

### [NIT] The package doc misplaces where the first records walk opens

**Location:** `internal/reportengine/doc.go:46`

**Issue:** `Compute` passes `landingWalkStart(opts)`, which is `Since` less `FreshDays`, to `newBattenIndex`.
`ComputeTiming` passes `Since`.
"A week before its window" is accurate only for the second.

## Rejected

- Mutation survivor at `boardhistory.go:101` (`done[task.Slug] = true`): dropping it only appends a duplicate one-commit done span for a slug with two done entries at one commit, which `doneAfter` reads the same way. It is an equivalent mutant, not a missing check.
- The affected-set lens's claim that `doc.go:46`'s "walks the records branch once" contradicts the widening: the next sentence says the walk widens, so the paragraph reads coherently. Only the window offset is kept, as F11.
- The scope lens's separate MEDIUM `scope` finding on the board floor after a full-history fallback is the same defect as F3, merged there.
- The correctness lens's LOW on `labelsAt` and the assistant's MEDIUMs on checkpoints and pruned labels repeat F5, merged there.

## Review coverage

- Range `2dbfdc314..HEAD`: three commits.
  `d067dad6d`'s `Root-Cause:` trailer names the unwindowed board and records walks and the caches of every parsed board and status, which the diff removes.
  No `Pattern-Departure:` or `Loosened-Check:` trailer is needed, since no existing test is loosened.
- Sweep: cost reads at tag commits and calls `ComputeTiming`, raddle reads the board tip, and replies takes supplied entries.
  `taskCommitBelow`'s paged walk stops at the pair's opening commit.
  None of these is an unwindowed walk, a per-commit read where one would do, or an all-revisions cache.
- Timing: sharing one index between landed and in-flight runs changes neither set's runs or spans, because a widen during `selectRuns` only adds commits.
- Memory: `boardTasksAt`'s tasks are dropped after `add`, which keeps cloned labels and index spans.
  `battenStatusReader` lives for one `snapshotsOfRun` call, and rows keep only their own span's snapshots.
- Merge danger: the diff changes no record, board or config format and pushes nothing, so the `two-way` door and `report` blast radius hold.
- `lym board get` returned an invalid-json error, so the decision record's Goal served as the board entry.
