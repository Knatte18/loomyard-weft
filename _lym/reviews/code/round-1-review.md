---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: internal/creelengine/sweep.go:134-148
    summary: "Mail stored for an unlisted hub orch is returned undelivered by the removed-holder sweep about 40 s after the post, so it does not wait for the orch (Goal item 3) and dobby has already cleared its request"
    fix: "In internal/creelengine/sweep.go make the sweep leave a holder that is the fixed name of an orch of this hub (Store.isHubOrchName) alone while no strand of that name is listed: in sweepOnce, when a holder is absent from `present` and s.store.isHubOrchName(holder), reset state.absent[holder] and continue before the miss count, so releaseGoneHolder never runs for it and its unread mail stays in the pool until the strand is listed. Update the sweepSubjects and releaseGoneHolder doc comments to say a hub orch's unread mail is never returned as a gone holder's. Add a row to TestSweep_RemovedHolders in internal/creelengine/deliver_test.go that posts from a program to an unlisted hub orch, runs the sweep past goneAfterSweeps, asserts the message is still unread and not journaled undelivered, lists the orch strand, and asserts the strand reads the line."
  - id: F2
    severity: MEDIUM
    class: design
    origin: handler
    location: internal/dobbyengine/landnotice.go:102-120
    summary: "takeDoneEntries takes up a done flagged entry on every machine whose dobby serves the parent worktree, ignoring the entry's owner machine, so a machine that did not run the landing can send a second line to its own orch and clear the shared request"
    fix: "In internal/battencli/dobbyreads.go DobbyTasks read the effective owner (owners.Effective(task.Slug, parents.Of)) for an entry carrying notify_on_land as well as for a released task, so TaskView.OwnedElsewhere is set for it, and update the TaskView.OwnedElsewhere comment in internal/dobbyengine/starts.go. In internal/dobbyengine/landnotice.go takeDoneEntries skip a task whose OwnedElsewhere is true, so only the owner's dobby sends the line and clears the request, per PATTERN-owner-machine. Add a landnotice_test.go row with a done flagged entry of this worktree that is owned elsewhere and assert no line and no clear, and extend the dobbyreading_test.go `landed` row to assert OwnedElsewhere for an entry another machine owns."
  - id: F3
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/dobbyengine/landnotice_test.go:70-141
    summary: "The Goal asks for the notice shown end to end against a live orch once, in a test or by hand, and says which in the change description; no test joins dobby, creel and an orch strand, and the summary says it was not done by hand either"
    fix: "Add one test that drives the real creelengine.Store with its scheduler and sweep over creelfake delivery and wires a dobby daemon's Deps.Notice to post through it as the production wiring does (a program sender to the orch's fixed name `<shortname>:orch`). With the orch strand unlisted, run a landing tick and several sweeps, then list the orch strand and assert the strand reads `[dobby notice] <slug>: landed as <commit>` once and the board request is cleared. Put it in the package whose tier fits the lowest tier that can hold it (tier 0 over fakes). Then replace the 'End to end against a live orch' section of _lym/landing/summary.md with a statement that names the test as the end-to-end demonstration, or, if a hand run is done instead, states that and its result."
  - id: F4
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/dobbyengine/landnotice.go:149-179
    summary: "No test row fails ClearNotifyOnLand, fails the beam.yaml read, or carries an owed line across a restart, so the no-repost-after-a-failed-clear ordering, the modeRead gating and the restart claim in the summary are unpinned"
    fix: "In internal/dobbyengine/landnotice_test.go add fields failClears (clears that fail before the seam succeeds) and failConfigOnTick to the TestLandNotices table, and rows for: a requested run whose clear fails once (want one post, two clear calls, empty memory at the end); a mode read that fails on the landing tick (want no line until the read recovers, then one line, and no done entry taken up while it fails); and a failed post followed by a rebuilt daemon over the same state record (want the owed line posted once by the new daemon). Keep each as a row of the existing table, not a new test function."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/dobbyengine/landnotice.go:171-177
    summary: "With Deps.ClearNotifyOnLand nil a requested done entry is posted, its memory dropped and the entry retaken at the next tick, so the line repeats every tick"
    fix: "In internal/dobbyengine/landnotice.go landNotices, run takeDoneEntries only when d.deps.ClearNotifyOnLand is non-nil (`if tasksRead && modeRead && d.deps.ClearNotifyOnLand != nil`), and say in the Deps.ClearNotifyOnLand comment in internal/dobbyengine/loop.go that a nil seam also leaves done entries untaken."
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/battencli/dobbyreads.go:287-296
    summary: "A parents.Of error on a done flagged entry aborts the whole DobbyTasks read, so one entry whose parent cannot be read blocks every start of the tick for as long as the flag stays on it"
    fix: "In internal/battencli/dobbyreads.go DobbyTasks, for an entry that is not released (a done entry carrying notify_on_land) log the parents.Of error with logger.Warn and leave view.Parent empty instead of returning it, keeping the abort for a released task. Add a dobbyreading_test.go row for a failing parent read of a done flagged entry that asserts the other tasks are still returned."
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/boardcli/cli.go:253-259
    summary: "The lym board prune help still says it removes every done task, though a done entry carrying notify_on_land now survives, and the prose of the orch role, the overview and the board skill that describe prune say the same"
    fix: "In internal/boardcli/cli.go change the prune Short and Long text and the --apply flag help to say that a done entry still carrying notify_on_land is kept until its dobby has sent the line and cleared the field. Make the same one-clause correction where the prose says prune removes done entries: docs/overview.md (the Land row paragraph near line 721), contracts/orchrole/orch-role.md (near line 193) and plugins/ly/skills/board/SKILL.md (near line 185)."
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/boardcli/cli.go:1226-1233
    summary: "landedNotice fires on any upsert that returns a flagged done entry, so an edit that never touches notify_on_land claims the run already landed and that a line is coming, which is also false under land_notice off"
    fix: "In internal/boardcli/cli.go pass the upsert payload to landedNotice and return a notice only when fields[\"notify_on_land\"] == true and the task is done. Word it so it does not promise a line under land_notice off, for example by ending `its parent's dobby sends the landing line at its next tick, or only clears notify_on_land when land_notice is off`. Update stepNotifyOnLandOnALandedEntry in internal/boardcli/verbs_test.go to the new text and add an assertion that a title-only upsert of the flagged done entry returns no notice."
  - id: F9
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/creelengine/verbs_test.go:202-211
    summary: "The new creel row covers only the three-segment orch name, so the prime's `<shortname>:orch` form, a mistyped slug and the Forward path that also calls checkReceiverLocked have no case"
    fix: "In internal/creelengine/verbs_test.go extend the Post_ToAnOrchOfTheHubWithNoStrandWaitsForItsStrand row, or add cases to its table, with the prime's `ly:orch` posted while unlisted and read once listed, and a Forward of mail to an unlisted hub orch; keep the other-hub and orch-thread refusals."
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/dobbyengine/landnotice_test.go:49-58
    summary: "The row AFailedPostIsRetriedOnTheNextTick is now covered by AFailedPostIsOwedUntilItGoesOut and AnOwedLineOutlivesTheRunsListing, which both fail a post and assert the retry"
    fix: "In internal/dobbyengine/landnotice_test.go delete the AFailedPostIsRetriedOnTheNextTick row; AFailedPostIsOwedUntilItGoesOut (failPosts 4) and AnOwedLineOutlivesTheRunsListing (failPosts 1, wantPosts 2) keep the retry pinned."
  - id: F11
    severity: NIT
    class: design
    origin: lens:code-correctness
    location: internal/dobbyengine/landnotice.go:108-117
    summary: "A done entry with no listed run gets the line `landed, commit unknown` even when it was marked done by hand or is a plain project, which never landed"
    fix: "In internal/dobbyengine/landnotice.go takeDoneEntries, leave the wording as it is but state in the function comment and in the landing-notice list of internal/dobbyengine/doc.go that a done entry is read as landed whatever set its status, so the line can name a landing that a hand-marked entry never made."
---

### [BLOCKING] Unread mail for an unlisted hub orch is returned by the sweep

**Location:** internal/creelengine/sweep.go:134-148 with internal/creelengine/pool.go (`isHubOrchName`, `checkReceiverLocked`)
**Issue:** The new receiver rule stores a post to `<shortname>:orch` whether or not a strand of that name is listed, with the orch name as the message's holder. `sweepSubjects` counts that holder because `isStrandName` accepts an orch name and the message is unread (`sweep.go:184`). `sweepOnce` finds no strand of that name, and after `goneAfterSweeps` (2) misses, about 30 s plus the 10 s confirm, `releaseGoneHolder` runs. It sends the unread message back to its sender; the sender is dobby's program name, which `isStrandName` rejects, so the message is journaled undelivered (`sweep.go:222-224`). Goal item 3 asks that a line the orch cannot take wait in creel and reach the orch when it returns, and the summary and both `doc.go` files say it stays unread until the strand is listed. It does so only for about 40 s. Dobby's post succeeded, so it has cleared the request and dropped its memory: the notice is lost silently. The new creel row (`verbs_test.go:202`) lists the strand by hand and never runs a sweep, so no test sees it. I read the sweep and the delivery path; I did not run a live creel daemon.

### [MEDIUM] The done-entry step ignores the entry's owner machine

**Location:** internal/dobbyengine/landnotice.go:102-120, internal/battencli/dobbyreads.go:287-300
**Issue:** `takeDoneEntries` takes up every done flagged entry whose parent is this dobby's worktree. The board is shared between machines, and `TaskView.OwnedElsewhere` is read only for a released task, so the step has no owner check. A second machine whose prime dobby serves the same parent name sees the flagged entry too, sends its own line to its own orch, and clears the shared request, while the owner's machine also sends one. That breaks the one-line-per-run claim and the owner rule `PATTERN-owner-machine` states for acting on an entry. The run path avoided this because run records are local to the machine that ran them. I traced this from the code and the pattern; no test or run shows a second machine.

### [MEDIUM] The end-to-end demonstration the Goal asks for is not made

**Location:** internal/dobbyengine/landnotice_test.go:70-141, _lym/landing/summary.md
**Issue:** The Goal's closing demand is to show the notice end to end against a live orch once, in a test or by hand, and to say which in the change description. The summary says it was not done, and the dobby tests run over fakes while the creel test has no dobby. The demand is unmet. A test that joins the dobby `Notice` path, creel's store and sweep, and an orch strand would also have caught the first finding.

### [MEDIUM] Failed clear, failed mode read and restart have no test rows

**Location:** internal/dobbyengine/landnotice.go:149-179, internal/dobbyengine/landnotice_test.go:86-90
**Issue:** The harness's `ClearNotifyOnLand` always succeeds and `LoadConfig` never fails, and no test builds a second daemon over the same state record. The ordering the focus directive names (a failed clear after a successful post must not repost, and the clear is retried) is therefore unpinned, as is the `modeRead` gating in `rememberRuns` and `takeDoneEntries` and the summary's claim that an owed line survives a restart and a re-exec. Each is a behavior the diff adds with no covering row.

### [LOW] A nil ClearNotifyOnLand makes the line repeat every tick

**Location:** internal/dobbyengine/landnotice.go:171-177
**Issue:** With the seam nil, `settleLandNotices` posts the line, skips the clear and deletes the memory. The entry stays done and flagged with no memory, so `takeDoneEntries` retakes it at the next tick. Production always wires the seam (`internal/beamcli/dobbydeps.go:171`), so only a nil seam reaches this, but the old "nil clears nothing" contract no longer terminates.

### [LOW] One unreadable parent aborts the whole task read

**Location:** internal/battencli/dobbyreads.go:287-296
**Issue:** `DobbyTasks` returns the `parents.Of` error for a done flagged entry as it does for a released task, so the read of every task fails and the tick's start step fails for every slug. A released task is short-lived; a flagged done entry now stays until its line is sent, so the failure lasts as long as the flag does. `dobbyParentRuns` warns and skips such a read per run instead.

### [LOW] The prune help and the prose that describes prune are stale

**Location:** internal/boardcli/cli.go:253-259
**Issue:** The `prune` `Short` and `Long` text and the flag help say it removes every done task. A done entry carrying `notify_on_land` now survives, which is what an operator will wonder about after `prune --apply`. The same statement sits in `docs/overview.md`, `contracts/orchrole/orch-role.md` and `plugins/ly/skills/board/SKILL.md`; the diff updates only the Go doc comments and `boardengine/doc.go`.

### [LOW] The already-landed notice fires on edits that do not touch the field

**Location:** internal/boardcli/cli.go:1226-1233
**Issue:** `landedNotice` tests the returned task only, so a title-only upsert of an already flagged done entry prints that its run already landed and that a line is coming. Under `land_notice: off` no line comes, only a clear.

### [LOW] Creel test covers one orch name form

**Location:** internal/creelengine/verbs_test.go:202-211
**Issue:** The row posts to `ly:gone:orch` only. The prime's `<shortname>:orch` form, which dobby's notice uses for the prime, and the Forward path (`verbs.go:840`) that also calls `checkReceiverLocked` have no case.

### [LOW] A retry row is now redundant

**Location:** internal/dobbyengine/landnotice_test.go:49-58
**Issue:** `AFailedPostIsRetriedOnTheNextTick` fails one post and asserts the retry. `AnOwedLineOutlivesTheRunsListing` fails one post and asserts the retry, and `AFailedPostIsOwedUntilItGoesOut` fails four. Both cover its behavior.

### [NIT] A hand-marked done entry is noticed as landed

**Location:** internal/dobbyengine/landnotice.go:108-117
**Issue:** For a done entry with no listed run the line reads `landed, commit unknown`. `StatusDone` is documented as also meaning marked done by hand, and a plain project can be done with no run, so the line can name a landing that never happened.

## Rejected

- Lens code-affected-set F3 (a landed beam member whose origin record is gone falls to the prime's dobby): the fork did not trace whether teardown removes the record before the next tick, and `ParentOf` answers through the board's beam chain by design, so no defect is shown.
- Lens code-correctness item 10 (a failed `updateLandNotice(Line="")` after a successful post can repeat the line): this is at-least-once delivery on a state-write failure, which the design accepts and the code logs.
- Lens code-tests, a flagged done member of a done project and a survivor's `depends_on` or `order` naming a kept entry: `Store.Prune` leaves those rules unchanged and `TestNotifyOnLand` pins the kept and dropped entry, so another row would repeat covered behavior.
- Lens code-tests, `settleLandNotices` with an unreadable state record: the path only logs and returns, and an extra row would pin a log line.
- Lens code-affected-set item 6 (`"done"` literals not migrated to `StatusDone`): `PATTERN-named-constants` covers names that can go stale, and the diff's `isDone` rewrite is enough.
