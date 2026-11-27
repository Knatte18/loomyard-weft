---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/dobbyengine/config.go:149
    summary: "New text names config keys bare, and the two new gate-hold keys are repeated string literals instead of named constants (PATTERN-config-key-names, PATTERN-code-conventions)"
    fix: "In internal/dobbyengine/keynames.go declare gateHoldFactorKey = \"gate_hold_factor\" and gateHoldMinKey = \"gate_hold_min\", and use them in ConfigSpec's Categories and in both new LoadConfig refusals (keyName(const) and the --set argument). Write every config key that a comment or doc added or changed by this diff names in full: beam.run_watch, beam.gate_hold_factor, beam.gate_hold_min, beam.land_notice, beam.pull_request, beam.autonomy, and gate.tiers.<tier>.timeout_min for timeout_min. Sites: config.go (the Config, GateHoldFactor, LoadConfig and CheckLocalValues comments), doc.go (the per-task bullet, The run watch and The gate-hold notice sections), template.yaml (the gate_hold_factor and gate_hold_min trailing comments), tasksetting.go (file header and LookupRunWatch), runwatch.go (file header, RunWatchDeps.Level, RunWatchState.Level and LandNotice), gatehold.go (GateHolder.TierTimeout), internal/beamsetting/beamsetting.go (package doc and the RunWatchValues comments). CLI arguments such as --set run_watch= stay as they are."
    origin: handler
  - id: F2
    severity: BLOCKING
    class: consistency
    location: internal/beamcli/dobbydeps_integration_test.go:776
    summary: "The new and changed beamcli tests hand-write other modules' config files with os.WriteFile, which PATTERN-test-stand-ins forbids"
    fix: "In TestDobbyGateHolders write the tier1 declaration with configengine.Set over gateslot.ConfigSpec() at configengine.LayerDefault (as internal/gateslot/config_test.go does) instead of os.WriteFile, and provoke the load failure by removing the gate.yaml file instead of writing `slots: 0`; keep the assertion that the error names the gate config load. In TestHubDobbyDeps_LandNoticeReadsTheRunsWorktreeLayer set land_notice and run_watch in the beam worktree through configengine.Set over dobbyengine.ConfigSpec() instead of writing the bytes of beam.yaml."
    origin: handler
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/dobbyengine/gatehold_test.go:48
    summary: "No test has a holder whose worktree is the task worktree itself, the production case, so dropping the equality arm of underWorktree passes every row"
    fix: "Add a TestGateHolds row whose holder Worktree is exactly /hub/task and assert that one notice posts."
    origin: lens:code-tests
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/dobbyengine/runwatch_test.go#TestRunWatch
    summary: "Batten's reason as the fallback and the child's reason taking precedence have no test, since no RunView in the table sets Error"
    fix: "In BattenBlockedAtLandOverADoneChildPostsItsLine set RunView.Error and assert the line carries it; add to the same row, or one beside it, a tick where both the child's Error and RunView.Error are set and assert the child's reason is the one in the line."
    origin: lens:code-tests
  - id: F5
    severity: MEDIUM
    class: design
    location: internal/dobbyengine/runwatch_test.go#TestRunWatch
    summary: "Second-stop detection is pinned only through the child's folded repeats; batten's episode and the child's history length have no row"
    fix: "Add a row where batten stays blocked at one row while RunView.HistoryLen or LastRepeats rises and assert a second line, and one where the child stays blocked at one row while only ChildReading.HistoryLen rises and assert a second line."
    origin: lens:code-tests
  - id: F6
    severity: MEDIUM
    class: design
    location: internal/dobbyengine/donerun_test.go#TestClearDoneRuns
    summary: "Nothing tests that a watch step which cannot use the state record keeps the tick from clearing done runs"
    fix: "Add a TestClearDoneRuns case that makes runWatch's real read or write of the state record fail (for example an unwritable record directory through permkit.Chmod), runs runWatch then clearDoneRuns, and asserts ClearDoneRun is not called; do not set watchRecordFailed by hand."
    origin: lens:code-tests
  - id: F7
    severity: MEDIUM
    class: design
    location: internal/beamcli/dobbydeps_integration_test.go#TestDobbyGateHolders
    summary: "The hub wiring of the gate-hold notice is pinned only for Tier and TierTimeout; the mapped Worktree, PID, Started, Marker, Slot and Site and the wired TaskWorktree have no test"
    fix: "Assert every field of each mapped holder in TestDobbyGateHolders (acquire with a distinct Worktree, Site and Marker), and in an existing hubDobbyDeps test over a real pair assert that deps.GateHolds.TaskWorktree(slug) equals the pair's worktree path."
    origin: lens:code-tests
  - id: F8
    severity: LOW
    class: design
    location: internal/dobbyengine/runwatch.go:235
    summary: "A child write between the run listing and the watch's child read pairs a newer child state with an older derived state, so one stop can post two lines"
    fix: "In readWatchedRun, when the child is found but child.Modified differs from run.ChildStatusModified, keep the last child part and leave StatusSeen and ChildSeen unchanged, so the next tick reads the child against a fresh derived state. Add a TestRunWatch row for it."
    origin: lens:code-correctness
  - id: F9
    severity: LOW
    class: design
    location: internal/dobbyengine/runwatch.go:235
    summary: "While no child status has been read, PairDir is empty, so batten's reason renders task-worktree paths absolute, against D3"
    fix: "Have readNoticeChild (internal/battencli/dobbynotice.go) return PairDir whenever the task worktree is present, also with found false; in readWatchedRun take a non-empty child.PairDir after every successful read; and clear PairDir in WatchReading.compared so that learning it alone owes no line. Add the case to the AReasonIsRelativeToThePairAndOnOneLine row."
    origin: lens:code-scope
  - id: F10
    severity: LOW
    class: design
    location: internal/dobbyengine/gatehold.go:157
    summary: "On Windows MarkedPIDs always returns errors.ErrUnsupported, so every tier gate-hold notice drops the holder tree it had read and says the tree is unreadable"
    fix: "In gateHoldProcesses, when MarkedPIDs returns an error matching errors.ErrUnsupported, return the holder tree with no marked processes; keep today's unreadable answer for every other error. Add a TestGateHolds row with that error asserting that the child PIDs are named."
    origin: lens:code-correctness
  - id: F11
    severity: LOW
    class: consistency
    location: internal/battencli/cleanup.go:683
    summary: "lym batten cleanup help still says the dobby removes a done run once it counted the landing and decided the landing notice, omitting the new watch holds"
    fix: "Extend the sentence to say the dobby also waits until it owes no run watch line for the run and has read the run's beam.run_watch."
    origin: lens:code-affected-set
  - id: F12
    severity: LOW
    class: design
    location: internal/dobbyengine/runwatch_test.go#TestRunWatch
    summary: "No row shows that a status rewrite changing only the time or the reason text owes no line under rows"
    fix: "Add a row at level rows that calls writeChild with the same state and row and a different Error after a first tick, and assert no second line."
    origin: lens:code-tests
  - id: F13
    severity: LOW
    class: design
    location: internal/dobbyengine/gatehold_test.go#TestGateHolds
    summary: "Nothing fails if the run watch's whole-map rewrite loses the noticed gate holds, since every once-per-hold row runs with the watch unwired or ticks once"
    fix: "In AWatchLevelOfOffStillGetsTheNotice tick again after the notice, restart the dobby, tick once more, and assert there is still one notice."
    origin: lens:code-tests
  - id: F14
    severity: LOW
    class: design
    location: internal/dobbyengine/gatehold_test.go#TestGateHolds
    summary: "No row shows that the key of an ended hold leaves GateHolds"
    fix: "In AnotherProcessOrALaterStartIsANewHold assert that the state record's Watches[\"task\"].GateHolds holds only the current hold's key."
    origin: lens:code-tests
  - id: F15
    severity: LOW
    class: design
    location: internal/dobbyengine/runwatch_test.go#TestRunWatch
    summary: "No row changes only batten's status time and expects the child to be read again (D1)"
    fix: "Extend AChildChangedWithNoStatusWriteIsNotRead with a tick that changes only RunView.StatusModified and assert ReadChild is called once more."
    origin: lens:code-tests
  - id: F16
    severity: LOW
    class: design
    location: internal/dobbyengine/gatehold_test.go#TestGateHolds
    summary: "The largest-tier-timeout rule for a hold of several slots has no row with differing timeouts"
    fix: "In TwoSlotsOfOneProcessAreOneHoldFromItsEarliestStart give the two slots different TierTimeout values, the larger on the second, and assert the bound in the line follows the larger."
    origin: lens:code-tests
  - id: F17
    severity: NIT
    class: design
    location: internal/dobbyengine/runwatch_test.go#TestRunWatch
    summary: "The dobby log line of a posted watch line has no assertion"
    fix: "In one TestRunWatch row use passHarness.recordLog and assert the line `run watch task: <line>`."
    origin: lens:code-tests
  - id: F18
    severity: NIT
    class: design
    location: internal/dobbyengine/runwatch_test.go#TestRunWatch
    summary: "A failing StatusExists keeping an unlisted run's entry has no row"
    fix: "Add a row beside the AnUnlisted rows whose StatusExists returns an error and assert the memory is kept."
    origin: lens:code-tests
  - id: F19
    severity: NIT
    class: design
    location: internal/dobbyengine/config.go:148
    summary: "gate_hold_factor of .nan or .inf passes the range check, and the converted bound then notices every tier hold at once"
    fix: "Refuse a NaN or infinite GateHoldFactor in the same LoadConfig check, with a message saying it must be a finite number that is not negative, and add a TestLoadConfig row."
    origin: lens:code-correctness
  - id: F20
    severity: NIT
    class: consistency
    location: internal/gateslot/pool.go:84
    summary: "Holder.Marker's doc says every process the tier run starts carries the entry, but a nested lym gate test replaces it for the processes it starts"
    fix: "Add one sentence to the Marker field doc saying that a nested lym gate test run inside the slot replaces the entry for the processes it starts, so they do not carry this marker."
    origin: handler
---

### [BLOCKING] Config keys named bare, and new keys as repeated literals

**Location:** `internal/dobbyengine/config.go:149`, with the comment sites listed in the fix.

**Issue:** `PATTERN-config-key-names` requires a config key named in a message, comment or doc to be written in full, and a message to render it through the key-name helper from named constants.
The two new refusals call `keyName("gate_hold_factor")` and `keyName("gate_hold_min")` with literals, and each literal appears again in the `--set` argument and in `ConfigSpec`, which `PATTERN-code-conventions` also forbids.
The new comments and `doc.go` sections name `run_watch`, `gate_hold_factor`, `gate_hold_min`, `land_notice` and `timeout_min` bare.
Surrounding code does the same, and the board holds a later sweep task for it, but the rule binds new text now and the constraint wins over the surrounding convention.

### [BLOCKING] Tests hand-write other modules' config files

**Location:** `internal/beamcli/dobbydeps_integration_test.go:776`, `:801` and `:830`.

**Issue:** `PATTERN-test-stand-ins` says a config file is written by `configengine.Set` over its module's template, "the test edits no byte of either", and no change adds a stand-in the rule forbids.
`TestDobbyGateHolders` writes `gate.yaml` twice with `os.WriteFile`, once as malformed content.
`TestHubDobbyDeps_LandNoticeReadsTheRunsWorktreeLayer` has its hand-written `beam.yaml` extended with `run_watch: rows`.
Card 7 asks for the hub gate config to declare the tier but does not ask for it to be written by hand, so the constraint can be met.

### [MEDIUM] The task worktree itself is never matched in a test

**Location:** `internal/dobbyengine/gatehold_test.go:48`

**Issue:** The fake holders work in `/hub/task/sub` or `/hub/task-b`.
A tier run records `location.WorktreePath()` (`internal/gatecli/run.go:828`), so the equality arm of `underWorktree` carries the ordinary case with no test.

### [MEDIUM] Reason selection untested

**Location:** `internal/dobbyengine/runwatch_test.go`

**Issue:** No listed `RunView` sets `Error`.
Dropping `BattenError: run.Error` or the fallback in `watchLine` passes, and a batten block would then mail no reason (D3).

### [MEDIUM] Batten's episode stamp untested

**Location:** `internal/dobbyengine/runwatch_test.go`

**Issue:** Only the child's `LastRepeats` is varied.
Deleting the batten arm of the episode switch, or the child's `HistoryLen` from its sum, passes every row.

### [MEDIUM] Failed-record clear hold untested

**Location:** `internal/dobbyengine/donerun_test.go`

**Issue:** Card 3 asks that a watch step which failed to record keeps the tick from clearing.
Removing `d.watchRecordFailed ||` from `clearDoneRuns` passes.

### [MEDIUM] Gate-hold hub wiring mostly unasserted

**Location:** `internal/beamcli/dobbydeps_integration_test.go`, `TestDobbyGateHolders`

**Issue:** The test reduces each holder to tier and timeout, and the engine tests use a fake.
Dropping `Worktree`, `PID`, `Started` or `Marker` from the mapping passes everything while the notice stops working.

### [LOW] Derived state and child state read at different moments

**Location:** `internal/dobbyengine/runwatch.go:235`

**Issue:** `DobbyRuns` derives the state and stats the child status early in the tick, and the watch reads the child after the passes and notices.
A child that blocks in between posts `running; ... child blocked`, and the next tick's `halted` posts a second line for one stop.
The window is narrow: the child status must also have been written since the previous tick.

### [LOW] Batten reason keeps absolute paths without a child

**Location:** `internal/dobbyengine/runwatch.go:235`

**Issue:** `PairDir` is set only by a child read that finds a status.
A batten block before Run-Shed renders its task-worktree paths absolute, while D3 asks for relative paths.

### [LOW] Windows loses the holder tree

**Location:** `internal/dobbyengine/gatehold.go:157`

**Issue:** `internal/proc/envmarker_windows.go` returns `errors.ErrUnsupported` for every call, and every tier holder records a marker.
The notice then names no child process, although D6 asks for the holder's tree and the tree was read.
Card 6's wording for a failed scan stays in force for real failures.

### [LOW] Cleanup help is stale

**Location:** `internal/battencli/cleanup.go:683`

**Issue:** The done-run clear now also waits on the run watch, which the help does not say.

### [LOW] Time-only and reason-only changes untested under rows

**Location:** `internal/dobbyengine/runwatch_test.go`

**Issue:** If `compared` stripped nothing, `rows` would mail on every status rewrite and no row would fail.

### [LOW] Gate-hold keys against the watch's rewrite untested

**Location:** `internal/dobbyengine/gatehold_test.go`

**Issue:** The only row that wires `RunWatch` ticks once, so losing `GateHolds` in the map rewrite would go unseen.

### [LOW] Pruning of ended holds untested

**Location:** `internal/dobbyengine/gatehold_test.go`

**Issue:** Removing the prune passes, and the keys of a live run then accumulate.

### [LOW] Re-read on a batten status write untested

**Location:** `internal/dobbyengine/runwatch_test.go`

**Issue:** Dropping the `StatusModified` clause of the re-read condition passes.

### [LOW] Largest timeout of a mixed hold untested

**Location:** `internal/dobbyengine/gatehold_test.go`

**Issue:** Every multi-slot hold in the table has equal timeouts.

### [NIT] Watch log line untested

**Location:** `internal/dobbyengine/runwatch_test.go`

**Issue:** The dobby log line card 3 names has no assertion.

### [NIT] Failing StatusExists untested

**Location:** `internal/dobbyengine/runwatch_test.go`

**Issue:** Card 3's "a failed or nil `StatusExists` keeps it" has no row for the failure.

### [NIT] Non-finite factor accepted

**Location:** `internal/dobbyengine/config.go:148`

**Issue:** `.nan` and `.inf` are not negative, so they pass, and the duration conversion yields a bound every hold is past.

### [NIT] Marker doc overstates its reach

**Location:** `internal/gateslot/pool.go:84`

**Issue:** Card 5 states the nested-run bound, and the field doc claims the opposite.

## Rejected

- code-correctness F5, a line posted again when the second record write fails or the dobby dies between post and write: recording before the post would lose lines instead, and D4's restart criterion concerns a reading already in the memory, which holds.
- The landed line's autonomy stamp is read after the task worktree is torn down, so it takes a parent's value: D5 names `orchcli.WithRunAutonomy`, whose stamp is the value read at the call.
- Runs owned by another machine are in `d.listedRuns` and so are watched: D1 names that list.
- Two concurrent holds of one run share one topic, so the later notice replaces the earlier unread one: D6 fixes the topic, and the finding would be against the answer key.
- Each marked process outside the tree costs one `Tree` scan of `/proc`: card 6 prescribes it, and it runs once per hold.
- `dobbyengine`'s new fake seams for holders, processes and the child reading: card 3 and card 6 prescribe them, and they hand the engine values of types it declares.
- `TestRunWatch`'s `landingView` marks a run at `Land` as past Land, which production does only from `Worktree-Teardown`: the behaviour each row pins depends on the `PastLand` flag alone.
