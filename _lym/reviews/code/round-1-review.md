---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/battencli/leftoverpairs.go:56
    summary: MarkKeptCancels writes the kept cancel intent before it knows the commit can go, so a refused or failed commit leaves the intent uncommitted and unpushed for good, because the next call skips a record whose Cancel is set
    fix: In MarkKeptCancels, call the commit deps' MergeActive check before shedrun.WriteKeptCancel and return its refusal without writing. Also make the selection take a run whose status already holds a finished keep cancel intent naming this machine while the prime's records HEAD lacks that status change, and commit and push it through commitCancelRecord the same way. A write whose commit then fails is retried at the next tick.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/dobbyengine/orphan.go#dueOrphans
    summary: The first sight is recorded before the orphan notice is posted, and a failed post is only logged, so an orphan whose notice never reached an orch is still torn down OrphanGrace after the unnoticed first sight, against D7's no earlier than the grace after that notice
    fix: In dueOrphans, post each fresh orphan's notice through d.deps.Notice and check its error before recording the slug's first sight. Record the first-sight time only for slugs whose notice posted, and leave a slug whose post failed out of the record, so the next probe tick notices it again. Add a TestOrphans row whose Notice seam fails once, asserting no first sight is recorded and no teardown starts until a notice has posted.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/pairteardown/leftover.go#RemoveHandedOverPair
    summary: RemoveHandedOverPair calls RemovePair with no session end, so the pair's reed session stays live while its worktree is removed, against PATTERN-pair-remote's order of owned processes, then reed session, then removal; the in-pair hand-over this kind exists for is the case whose session is live
    fix: In RemoveHandedOverPair, keep the EndSubjectProcesses call, then replace the td.RemovePair call with td.Run over the same Request (Slug set to slug, Local and Force set, Verdict fabricengine.EntryOwnedElsewhere) and context.Background(), so EndSession ends the pair's reed session before the local removal. Return the Run result's Removal, or change the return type to Result and update the caller in internal/beamcli/dobbydeps.go#tearDownLeftover.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/beamcli/dobbydeps.go#tearDownLeftover
    summary: No test pins the kind-to-request dispatch (Force for cancelled, Landed for landed, local mode for handed-over, the orphan route, ended through the reconcile) or the hub workspace regeneration, so D7's criterion that the hub workspace file no longer lists a pair after its teardown is unpinned
    fix: Add a test in internal/beamcli over a hubforge hub with a landed pair and a cancelled pair without keep. Call tearDownLeftovers with both kinds and assert the channel maps each slug to nil, both pairs are gone, the cancelled run's batten record is gone, the landed run's board entry reads done, and fabricengine.HubWorkspacePath no longer lists either pair.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/pairteardown/leftover.go#RemoveOrphanPair
    summary: Neither RemoveOrphanPair nor RemoveHandedOverPair is tested, so D7's criterion that an orphan's branches are archived, uncommitted files committed onto the archived tip, and that a failed archive removes nothing, is unpinned
    fix: Add a test in internal/pairteardown over a hubforge hub. An orphan pair with a local commit origin lacks and an uncommitted file is removed by RemoveOrphanPair; assert the owner archive tag on origin names a commit holding that file and the pair is gone. A row whose origin refuses the tag push asserts the error and the pair still standing. A RemoveHandedOverPair row asserts the local pair is gone with no origin branch deleted.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/battencli/leftoverpairs.go#DobbyOrphans
    summary: No test pins the orphan classification, so a board-named pair, a pair with a batten run directory, the prime or a prunable worktree misread as an orphan would be torn down after the grace unnoticed by any test
    fix: Add a table test of DobbyOrphans in internal/battencli over a hubforge hub holding a pair on the board, a pair with a batten run directory and no board entry, and a pair with neither. Assert only the last is listed, with its LastCommit and a ProcessLive that follows a runproc record registered under its slug.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/battencli/leftoverpairs.go#MarkKeptCancels
    summary: No test pins the pre-upgrade selection (state cancelled, no intent, pair standing here) or its commit and push, so D7's criterion that a pre-upgrade kept pair is listed as kept after the first dobby start is pinned only through a faked seam
    fix: Add a test in internal/battencli over a hubforge hub with a batten record in state cancelled with no cancel intent and its pair standing, and one whose pair is gone. Call MarkKeptCancels and assert it returns only the first slug and the record carries a finished keep intent naming this machine. Assert the record is committed on the prime's records side and DobbyRuns then reads the run PairKept.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/battencli/overviewdobby.go#DobbyLines
    summary: TestDobbyLines has no row with view.Kept, so D7's criterion that lym beam status lists a kept pair with its age is unpinned at the renderer, for the age and the unknown-age wording alike
    fix: Add two rows to TestDobbyLines in internal/battencli/overviewdobby_test.go. One has a view.Kept entry with a Since, the other a zero Since. Assert the exact lines, kept pair, cancelled <minutes> ago and kept pair, age unknown, each followed by the way forward naming lym fabric remove <slug>.
  - id: F9
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/pendingflags.go#PairPushDeferred
    summary: No test pins that a left pending flag reads as a deferred push and its absence as none, so D7's still pays a deferred anchor-change push is pinned only against the fake PushDeferred seam in TestResume
    fix: Add a table test of PairPushDeferred in internal/fabricengine over a hubforge pair with rows for no pending flag (false), a code-side flag (true) and a records-side flag (true). Write each flag at the path pendingFlagPath gives under RecordsLockDirPath(RecordsWorktree(pair)).
  - id: F10
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/dobbyengine/donerun.go:117
    summary: The new PairPresent skip in clearDoneRuns has no test row, though without it the clear would remove a landed run's records before the leftover teardown reads them
    fix: Add a row to the done-run clear test in internal/dobbyengine with a done run whose PairPresent is true, asserting ClearDoneRun is not called for it, and that a later tick with PairPresent false clears it.
  - id: F11
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/battencli/leftoverpairs.go:108
    summary: A PairLastCommit failure on one worktree fails the whole orphan listing, so one broken pair stops every orphan from being noticed or torn down, unlike an unresolvable worktree, which is skipped with a Warn
    fix: In DobbyOrphans, when fabricengine.PairLastCommit returns an error, log one logger.Warn naming the slug and the error and continue to the next worktree, as the unresolvable-worktree branch above it does.
  - id: F12
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/pairteardown/leftover.go#RemoveOrphanPair
    summary: RemoveOrphanPair re-checks nothing, so a pair put on the board, given a batten record or given a live process after the dobby's last orphan reading can still be torn down from the cached due list
    fix: At the start of RemoveOrphanPair, read the board at fabricengine.BoardDir(prime.HubPath), the batten run directory shedrun.RunDir(prime, slug) and runproc.List(fabricengine.RunProcDir(prime.HubPath), slug). Return an error naming the reason, with nothing archived or removed, when the board names the slug, the run directory exists or a process lives.
  - id: F13
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/orphan.go#PairLastCommit
    summary: PairLastCommit is untested, so a wrong reading of the newer of the two tips, or of a missing worktree as zero, would skip or wrongly allow an orphan teardown with no test failing
    fix: Add a test in internal/fabricengine over a hubforge pair whose code and records tips carry different commit times. Assert PairLastCommit returns the newer, and the code tip's time once the records worktree is removed.
  - id: F14
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/battencli/dobbyreads.go#DobbyRuns
    summary: No test pins DobbyRuns' reading of the new fields PairKept, KeptSince through keptSince, OwnedElsewhere, BoltHeld and HandOverMarked, which the engine tests set by hand
    fix: Extend the DobbyRuns test in internal/battencli with a run cancelled with keep whose finish has an EndedAt, and assert PairKept and KeptSince. Add a run whose hand-over mark fabricengine.WriteHandOverMark wrote, owned by another registered machine, and assert HandOverMarked and OwnedElsewhere. Add a run under a Bolt hold and assert BoltHeld.
  - id: F15
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/dobbyengine/orphan.go#dueOrphans
    summary: No TestOrphans row covers a failing Orphans seam or a failing record write, which keep the last reading and log once per episode
    fix: Add a TestOrphans subtest whose Orphans seam fails after a first good reading. Assert the due list from the good reading still stands, with one failure log, and that a later good reading recovers.
  - id: F16
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/dobbyengine/doc.go:584
    summary: The new kept-pairs bullet sits between the runs bullet and its continuation line, It also carries its last pass exit..., which now reads as describing the kept pairs
    fix: In internal/dobbyengine/doc.go, move the line beginning The kept pairs are the listed cancelled runs' pairs below the continuation line beginning It also carries its last pass exit, so the continuation follows the runs bullet it belongs to.
  - id: F17
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/pairteardown/doc.go
    summary: The package doc describes each removal entry point, Request.Local, HandOver and HandOverOwnedElsewhere, but not the two new exported removals RemoveHandedOverPair and RemoveOrphanPair
    fix: In internal/pairteardown/doc.go, beside the hand-over section, add a paragraph on RemoveHandedOverPair (ends the run's processes, then removes the local pair in local mode with force) and RemoveOrphanPair (archives the code and records tips with uncommitted files under owner archive tags, then tears the pair down leaving its branches on origin and settling no claim).
  - id: F18
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/tendercli/cli.go:215
    summary: The edited lift paragraph of the lym tender resume help runs one line to 141 characters while the rest of the help text wraps near 80 columns
    fix: In internal/tendercli/cli.go, rewrap the help paragraph that begins A lift also spawns what a pause deferred so each line stays near 80 columns like the surrounding help text.
---

### [MEDIUM] A kept cancel intent whose commit fails is never committed

**Location:** `internal/battencli/leftoverpairs.go:56`

**Issue:** `MarkKeptCancels` calls `shedrun.WriteKeptCancel` and only then `commitCancelRecord`, which refuses while a records merge is in progress and can fail its commit.
After such a failure the local status already holds `Cancel`, so the next call skips the run at the `run.status.Cancel != nil` guard and returns no error.
The dobby then counts the marking as done, and the intent never takes D7's synchronous push.

### [MEDIUM] An orphan can be torn down without its notice ever posting

**Location:** `internal/dobbyengine/orphan.go#dueOrphans`

**Issue:** `dueOrphans` writes the first-sight time inside `updateOrphanSights` and posts the notice after it through `d.post`, which only logs a failed post (`loop.go:453`).
A post that fails, for example with no live orch, still leaves the first sight recorded, so the grace runs from an unnoticed sight.
D7 asks for a teardown "no earlier than the orphan grace after that notice".

### [MEDIUM] The handed-over pair is removed without its reed session ending

**Location:** `internal/pairteardown/leftover.go#RemoveHandedOverPair`

**Issue:** `EndSubjectProcesses(prime, slug, false)` ends the processes registered under the slug, since `runproc.EndSubject` passes over the subject's own records.
`RemovePair` is then called with no `EndSession`, though its contract says it runs only after the session end.
`HandOver` leaves the pair to the parent's dobby precisely when the hand-over ran inside the pair, so the pair this kind removes is the one whose reed session is most likely still live.
PATTERN-pair-remote orders a teardown as owned processes, then reed session, then removal.

### [MEDIUM] The hub wiring's per-kind dispatch and workspace regeneration are untested

**Location:** `internal/beamcli/dobbydeps.go#tearDownLeftover`

**Issue:** No test exercises `tearDownLeftovers` or `tearDownLeftover`.
The request each kind builds (`Remote`, `Force`, `Landed`, local mode, the orphan route, the ended reconcile) and the call to `fabricengine.RegenerateHubWorkspace` are unpinned.
That leaves the acceptance criterion that the hub workspace file no longer lists a pair after its teardown without a test.

### [MEDIUM] The orphan and handed-over removals are untested

**Location:** `internal/pairteardown/leftover.go#RemoveOrphanPair`

**Issue:** Neither `RemoveOrphanPair` nor `RemoveHandedOverPair` has a test.
Untested are the archive of the tips with uncommitted files before removal, a failed archive removing nothing, and the local mode of the hand-over removal.
D7's "its branches archived" is unpinned.

### [MEDIUM] The orphan classification is untested

**Location:** `internal/battencli/leftoverpairs.go#DobbyOrphans`

**Issue:** No test reads `DobbyOrphans`.
A misclassification would put a live pair on the 24-hour teardown path with no test failing: a board-named pair, a pair with a batten run directory (D7's bound for a kept pair), the prime or a prunable worktree.

### [MEDIUM] The pre-upgrade kept-cancel marking is untested at the hub level

**Location:** `internal/battencli/leftoverpairs.go#MarkKeptCancels`

**Issue:** `TestKeptPairs` drives a fake `MarkKeptCancels` seam, and `TestRunIntents` pins `WriteKeptCancel` alone.
Nothing pins the selection of cancelled records with no intent whose pair stands here, or the commit and push that follow.
D7's criterion that such a pair is listed as kept after the first dobby start is therefore unpinned.

### [MEDIUM] The kept line of lym beam status is untested

**Location:** `internal/battencli/overviewdobby.go#DobbyLines`

**Issue:** `TestDobbyLines` has no row with `view.Kept`, so the kept line D7 asks for in `lym beam status` is unpinned at its renderer.
That covers the age, the unknown-age wording and the way forward.

### [MEDIUM] PairPushDeferred's reading of the pending flag is untested

**Location:** `internal/fabricengine/pendingflags.go#PairPushDeferred`

**Issue:** `TestResume` fakes the `PushDeferred` seam, so nothing pins that the real reader answers true for a left code or records pending flag and false with none.
That reading is what D7's "still pays a deferred anchor-change push" rests on.

### [MEDIUM] The done-run clear's new pair skip is untested

**Location:** `internal/dobbyengine/donerun.go:117`

**Issue:** `clearDoneRuns` now skips a done run whose pair is present, so the leftover teardown reads the landed run's records first.
No done-run test row sets `PairPresent`, so dropping the skip fails nothing.

### [LOW] One unreadable pair fails the whole orphan listing

**Location:** `internal/battencli/leftoverpairs.go:108`

**Issue:** A `fabricengine.PairLastCommit` error on any worktree, such as a repository with no commit, returns from `DobbyOrphans`.
Every other orphan then goes unnoticed and is never torn down while that pair stands.
An unresolvable worktree is skipped with a Warn instead.

### [LOW] The orphan removal re-checks nothing before it acts

**Location:** `internal/pairteardown/leftover.go#RemoveOrphanPair`

**Issue:** Between probe ticks `dueOrphans` returns the cached due list.
`RemoveOrphanPair` then runs with `Force` and an ended verdict, and re-reads neither the board, the batten run directory nor the slug's processes.
A pair claimed or put to use after the last reading can be torn down in that window.

### [LOW] PairLastCommit is untested

**Location:** `internal/fabricengine/orphan.go#PairLastCommit`

**Issue:** Nothing pins that it returns the newer of the code and records tips, or that it reads a missing worktree as no commit.

### [LOW] DobbyRuns' new fields are untested

**Location:** `internal/battencli/dobbyreads.go#DobbyRuns`

**Issue:** `PairKept`, `KeptSince`, `OwnedElsewhere`, `BoltHeld` and `HandOverMarked` decide every leftover kind.
The engine tests set them by hand, and no battencli test reads them from records.

### [LOW] The orphan reading's failure path is untested

**Location:** `internal/dobbyengine/orphan.go#dueOrphans`

**Issue:** The `TestOrphans` seams never fail.
The branch that keeps the last reading and logs once per episode on a failed `Orphans` call or record write has no row.

### [LOW] The kept-pairs view bullet splits the runs bullet from its continuation

**Location:** `internal/dobbyengine/doc.go:584`

**Issue:** The new bullet now sits between "Each carries batten's state and producer…" and "It also carries its last pass exit…".
The continuation line reads as describing the kept pairs.

### [LOW] The pairteardown package doc omits the two new removals

**Location:** `internal/pairteardown/doc.go`

**Issue:** The package doc describes `Request.Local`, `HandOver` and `HandOverOwnedElsewhere`.
It describes neither `RemoveHandedOverPair` nor `RemoveOrphanPair`, which the dobby now calls.

### [NIT] One overlong line in the lym tender resume help

**Location:** `internal/tendercli/cli.go:215`

**Issue:** The edited lift paragraph leaves one 141-character line in help text that otherwise wraps near 80 columns.

## Rejected

- code-correctness F3's claim that `EndSubjectProcesses(prime, slug, false)` ends none of the task's own processes is wrong: `runproc.EndSubject` ends the subject's own records (`end.go:115`), and the subject is the slug. The kept finding F3 rests on the missing reed session end alone.
- code-scope reported no finding; its trace of every change to a goal point is accepted.

The handler's own holistic pass found nothing beyond the lens findings.
The diff traces to the board entry's Delivers and Exports lines, keeps every teardown inside `internal/pairteardown`, and names the orphan grace once as `dobbyengine.OrphanGrace`.
The new functions all have production callers, and the test-only reader sits in `leftover_test.go`.
Each summary claim checked holds against the diff, including that the tender change is its own commit.
