---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/dobbyengine/runnotice_test.go (TestRunNotice_APIErrorRevive); internal/dobbyengine/runnotice.go:379
    summary: No test pins the notice memory (episode since, api-error revive streak) surviving a dobby re-exec through the state record
    fix: Add a restart row to TestRunNotice_APIErrorRevive in internal/dobbyengine/runnotice_test.go. Tick once so the first sight revives, replace the daemon with a fresh one over the same state file, then assert no second revive and a notice at the next sight of the same episode. Also assert the persisted RunState.Notice fields (APIReviveKey, APIRevives, APIReviveStatusTime, ChildState, Since) are read back.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/dobbyengine/runnotice.go:149-168
    summary: No test covers which runs the run notices visit - a cancel marker, a non-running batten state, an absent child status, a ReadChild error or a nil ReadChild
    fix: In internal/dobbyengine/runnotice_test.go add rows using run views with CancelMarker set and with a non-running State, plus ReadChild fakes that return found=false, return an error, and are nil; each expects no notice and, for the error, a tick that still succeeds.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/battenshed/innerrun_memory.go:199-202; internal/battencli/dobbynotice.go:171-174
    summary: RespawnCapReached reads the saved respawn count without the status-write reset, so a stale cap can name the cap in a driver-dead notice, and a memory read error hides every condition of the run
    fix: In readNoticeChild (internal/battencli/dobbynotice.go) treat an error from battenshed.RespawnCapReached as not capped and still return the child reading, logging the error at Debug. In RespawnCapReached (internal/battenshed/innerrun_memory.go) take the child status file's modification time and report false when it is after memory.RespawnStatusTime, matching the reset the row's pass applies before respawning.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: internal/dobbyengine/runnotice.go:43
    summary: dobbyengine redeclares respawnCap = 3 for the notice text while the capped reading comes from battenshed's own respawnCap
    fix: Drop the dobbyengine respawnCap constant and its use in noticeCondition. Have ChildReading carry the cap count (for example RespawnCap int set from the battenshed reader in internal/battencli/dobbynotice.go) or render the text from that field, so one constant in internal/battenshed/innerrun.go decides both the reading and the text.
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/shedrecipe/entries_batten.go:122; internal/beamcli/dobbydeps.go:78-90
    summary: Dropping the zero-to-30 default for notice_quiet_min means an explicit zero now reaches dobby as Quiet 0 and silently disables the quiet notice
    fix: In runShedDuration (internal/beamcli/dobbydeps.go) return fallback when the count is not positive, so a zero keeps the 30-minute default as before; or, if disabling is intended, say so in innerRunEntry's doc comment in internal/shedrecipe/entries_batten.go and in the batten recipe's comment on the key.
  - id: F6
    severity: LOW
    class: design
    origin: handler
    location: internal/battenshed/notice.go:471-482
    summary: resetText and resetStamp have no caller left in battenshed after the move, so they are dead code
    fix: Delete resetText and resetStamp, with their doc comments, from internal/battenshed/notice.go, and drop any import only they used.
  - id: F7
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/dobbyengine/runnotice.go:279,310
    summary: The zero quiet window that sends no quiet notice has no test row
    fix: Add a row to TestRunNotice_Conditions in internal/dobbyengine/runnotice_test.go that sets RunNotices.Quiet to zero with a long idle time and expects no notice, for both the no-run fallback and the agents-read path.
  - id: F8
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/battencli/dobbynotice.go:146-214
    summary: Only the absent-worktree path of readNoticeChild is tested; the field mapping and the lock fallback of noticeDriverAlive are not
    fix: Add a test in internal/battencli over a temp task worktree with a status file and a memory record that asserts State, Modified, NewestHistory, HistoryLen and RespawnCapped, and add rows for noticeDriverAlive with a held driver lock, a held run lock and an unreadable lock, each reading as alive.
  - id: F9
    severity: NIT
    class: scope
    origin: lens:code-tests
    location: internal/dobbyengine/runnotice.go:169-173
    summary: The cleanup of delivery attempts for runs no longer listed has no test
    fix: Add an assertion to the delivery retry test in internal/dobbyengine/runnotice_test.go that a failed notice's attempt entry is gone after its run leaves the listing, so a later run with the same slug and key starts at zero attempts.
---

### [MEDIUM] Notice memory across a re-exec is untested

**Location:** internal/dobbyengine/runnotice_test.go (TestRunNotice_APIErrorRevive); internal/dobbyengine/runnotice.go:379
**Issue:** The decision record's D3 and the acceptance criteria say the episode memory and the revive streak survive a dobby re-exec through the state record. `TestRunNotice_OncePerEpisode` replaces the daemon, but the driver-dead key is recomputed from the child's modification time, so the test passes with `saveNoticeMemory` removed. The streak fields are never checked across a restart.

### [MEDIUM] Run selection of the run notices is untested

**Location:** internal/dobbyengine/runnotice.go:149-168
**Issue:** D2 names a cancel marker, an absent child status and a non-running run as in no condition. The test row for a child that is not running sets the child's state, and its ReadChild always reports found, so the batten-state and cancel-marker skips, the absent status, a read error and a nil ReadChild have no row that fails when broken.

### [LOW] Stale respawn cap and a memory read error that hides a run

**Location:** internal/battenshed/innerrun_memory.go:199-202; internal/battencli/dobbynotice.go:171-174
**Issue:** `RespawnCapReached` returns the saved count. The row resets it on a later status write only in the pass about to respawn, so between the write and that pass a dead driver is noticed with the respawn-cap text although progress was made; the removed code read the count after that reset. An error reading the memory record returns not found from `readNoticeChild`, which sends no condition at all for the run, a dead driver included, until the record reads again.

### [LOW] Respawn cap declared twice

**Location:** internal/dobbyengine/runnotice.go:43
**Issue:** The notice text uses dobbyengine's own `respawnCap`, while whether the child is capped comes from battenshed's constant. A change to one leaves the text naming a number the reading did not use.

### [LOW] Explicit zero notice_quiet_min now disables the quiet notice

**Location:** internal/shedrecipe/entries_batten.go:122; internal/beamcli/dobbydeps.go:78-90
**Issue:** `innerRunEntry` used to turn a zero into 30 minutes; it now only validates non-negative, and nothing reads the key there. `runShedDuration` passes a zero through, so dobby gets `Quiet` 0 and sends no quiet notice. The shipped recipe sets 30, so only a hand-edited zero changes behaviour.

### [LOW] Dead helpers left in battenshed

**Location:** internal/battenshed/notice.go:471-482
**Issue:** `resetText` and `resetStamp` were used only by the usage-limit judgement that moved to dobbyengine, which has its own copies. Nothing in battenshed, tests included, calls them now.

### [LOW] Zero quiet window untested

**Location:** internal/dobbyengine/runnotice.go:279,310
**Issue:** The "zero sends none" contract of `RunNoticeDeps.Quiet` has no test row.

### [LOW] Notice child reads and the lock fallback largely untested

**Location:** internal/battencli/dobbynotice.go:146-214
**Issue:** Only the absent-worktree path of `readNoticeChild` is tested. The mapping of the status file and memory record into `ChildReading`, and the held-lock and unreadable-lock reads of `noticeDriverAlive`, have no test. The strand half is covered by the existing `driverAliveFrom` tests.

### [NIT] Delivery-attempt cleanup untested

**Location:** internal/dobbyengine/runnotice.go:169-173
**Issue:** Removing the loop that drops attempts for runs no longer listed would leave every test green.

## Rejected

- Fork note that a failed marker write repeats the notice each probe tick: batten's `saveEpisode` behaved the same, nothing regressed, and the doc comment says a lost marker costs at most one repeated notice.
- Fork note that a superseded episode's failed delivery entry stays until the slug leaves the listing: negligible, bounded by the run's life.
- The removals in `internal/testkit/envkit/envkit.go` and `internal/loomcli/wiring_seams_test.go` that the focus asks about: they follow from the removed `InnerRunDeps` seams and are licensed by the answer key.
- Declared PATTERN changes: the directive declares none, so nothing is owed.
- Orch stencil, `docs/overview.md`, package docs: no moved notice is still named `[batten notice]`, as D5 requires; the remaining `[batten notice]` mentions name only notices batten keeps.
