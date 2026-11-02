---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/landingshed/finalize.go:820
    origin: lens:code-correctness
    summary: a fixer turn that leaves HEAD at Baseline proceeds without recording Judged, so a later merge of parent commits is never rebased and the parent's lines count as the fixer's change
    fix: In judgeFixerChange, in the `rec.Baseline == head` branch, set `rec.Judged = head` and write the record with WriteFixerAttempts before returning proceed(), as letThrough does (or route that branch through letThrough). Add a row to the TestFinalize_GateAndFixer lock-freed table where the fixer makes no commit, the next merge brings parent commits, and LineChange is measured from the merged head.
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/landingshed/finalize.go:725
    origin: lens:code-tests
    summary: rebaseAfterParentMerge has no row for a no-op merge or for a merge whose TaskHead differs from Judged, so a rebase that always fired would hide a real fixer change from the caps unnoticed
    fix: In finalize_gate_test.go add rows to the lock-freed table with the countingResolver answering AlreadyUpToDate, and one where the merge's TaskHead differs from Judged; assert Baseline and Judged stay unchanged and that LineChange is measured from the old Baseline.
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/landingshed/finalize.go:844
    origin: lens:code-tests
    summary: the fix review frees the parent lock but no test reads the lock record while a review session runs, so a regression putting it back under the lock passes every test
    fix: In finalize_gate_test.go add a TestFinalize_GateAndFixer row whose fixer change is over the caps, and have the fix review session's seat read the lock record through the guard fixture's store; assert it is absent during the review, as the fixer-turn row does.
  - id: F4
    severity: LOW
    class: design
    location: internal/parentlock/acquire.go:76
    origin: lens:code-correctness
    summary: pause does not detect a closed changes channel, so a watcher whose events channel closes mid-wait makes every pause return after the floor and the backoff to the thirty-second ceiling is lost
    fix: In Waiter.pause receive with `case _, ok := <-changes` and, when !ok, return w.sleep(ctx, d); have pause report the closed channel to Acquire (or have Acquire set changes to nil) so later iterations use the plain sleep. Add a test row that closes the watch channel and asserts the next delay doubles.
  - id: F5
    severity: LOW
    class: design
    location: internal/landingshed/finalize.go:215
    origin: lens:code-correctness
    summary: a resumed Finalize starts with front=false, so a run that halted at the requeue wait after its fixer queues at the back instead of the head the decision gives a run back from its fixer
    fix: Let Call start its loop at front when the attempt record of the row's entry shows a fixer change let through (`Judged != ""`), reading the record through readAttempts, or state the bound in landingshed/doc.go next to the front-queue sentence. Pick one and add a test row for it.
  - id: F6
    severity: LOW
    class: decision
    location: internal/landingshed/doc.go:49
    origin: lens:code-affected-set
    summary: with the lock freed, a hub pause covering the run now holds the fixer's gate calls, which the lock-holder exemption in loomcli/landingdeps.go:543 skipped before; neither the code nor the docs say so
    fix: Add one sentence after the new lock-freeing paragraph in landingshed/doc.go stating that a pause covering the run holds the fixer's gate calls and the requeue wait, since the run holds no lock then; leave the exemption in loomcli/landingdeps.go as it is unless the parent decides otherwise through lym loom decision add.
  - id: F7
    severity: LOW
    class: design
    location: internal/landingshed/finalize.go:327
    origin: lens:code-tests
    summary: the requeue after the merge-in-required retry has no test row, so a fixer running inside the retry's mergeAndGate is not pinned to take the lock again before landing
    fix: In finalize_gate_test.go add a row where the first parent-side merge returns ErrMergeInRequired and the retry's gate fails, so the fixer runs; assert a requeue, a second lock take at the front and a landing.
  - id: F8
    severity: LOW
    class: design
    location: internal/landingshed/finalize.go:824
    origin: lens:code-tests
    summary: Judged is asserted only indirectly and only on the within-caps route, so the fix-review-approved and operator-accepted routes could drop it with no test failing
    fix: In finalize_gate_test.go assert the attempt record's Judged equals the task head after a fix review approval and after an acceptance, and that a later merge onto it moves Baseline.
  - id: F9
    severity: LOW
    class: design
    location: internal/parentlock/store.go:240
    origin: lens:code-tests
    summary: the order of two front tickets and a front waiter waiting behind an older front ticket have no test
    fix: In parentlock/acquire_test.go add a row with two front waiters queued behind a holder; assert the older one is granted first at the release and the younger stays queued.
  - id: F10
    severity: LOW
    class: design
    location: internal/parentlock/acquire.go:115
    origin: lens:code-tests
    summary: the watch-open failure Warn and the poll-only fallback have no test because fswatch.Watch is not injectable
    fix: Add a Waiter or Store seam that replaces the watch open, and a test row where it fails; assert the wait still grants on the poll and logs the Warn once.
  - id: F11
    severity: NIT
    class: consistency
    location: internal/parentlock/store.go:107
    origin: handler
    summary: the Covered field comment still says the queue head is the oldest live ticket of an uncovered run, but the head is now the first live ticket in front-first order
    fix: Change "the oldest live ticket of an uncovered run" in the Store.Covered field comment to "the first live ticket of an uncovered run", as TryAcquire's doc comment now reads.
---

### [MEDIUM] Fixer turn with no commit leaves Judged empty
**Location:** internal/landingshed/finalize.go:820
**Issue:** The first red verify sets Baseline to HEAD in runFixer. When the fixer fixes the failure without a commit, judgeFixerChange hits `rec.Baseline == head` and returns proceed() with Judged still empty. After the requeue the merge brings parent commits, rebaseAfterParentMerge returns early on `rec.Judged == ""`, and a passing verify then measures `LineChange(Baseline, head)` over the parent's commits. Over the caps this starts fix reviews or a halt for lines the fixer never wrote, the defect the Judged field was added to prevent.

### [MEDIUM] No-op and mismatched-head branches of rebaseAfterParentMerge untested
**Location:** internal/landingshed/finalize.go:725
**Issue:** Both table rows end in a merge that brings parent commits. A no-op merge, where Baseline must stay put, and a merge whose TaskHead is not Judged have no row. A rebase that fired on a no-op merge would set Baseline to HEAD and hide the whole fixer change from the caps, with every test still green.

### [MEDIUM] Fix review lock free untested
**Location:** internal/landingshed/finalize.go:844
**Issue:** The diff frees the lock before runFixReview and the doc.go text and summary claim it, but only the fixer's turn is checked against the lock record. Moving the review back under the lock passes every test.

### [LOW] Closed watch channel collapses the backoff
**Location:** internal/parentlock/acquire.go:76
**Issue:** `<-changes` on a closed channel returns at once, so each pause costs only the rest of the floor. The wait then wakes every second for as long as it lasts and never backs off to the thirty-second ceiling.

### [LOW] Front priority lost across a resume
**Location:** internal/landingshed/finalize.go:215
**Issue:** The loop starts with front=false on every Call. A run that halted at the requeue wait after its fixer, for instance at a pause, resumes into the ordinary queue and waits behind runs that queued while it fixed.

### [LOW] Pause now holds the fixer's gate calls, unstated
**Location:** internal/landingshed/doc.go:49
**Issue:** holdsParentLock reads HeldBy(slug), false once freeLock released the record, so a hub pause covering the run holds the fixer's own gate calls and the requeue wait, where the lock-holder exemption let them through before. Nothing in the code or the docs states the change.

### [LOW] Requeue after the merge-in-required retry untested
**Location:** internal/landingshed/finalize.go:327
**Issue:** No row runs a fixer inside the retry's mergeAndGate, so the second `lockFreed` check is unpinned.

### [LOW] Judged covered on the within-caps route only
**Location:** internal/landingshed/finalize.go:824
**Issue:** The approved and accepted routes through letThrough have no assertion on Judged or on a later Baseline move.

### [LOW] Front ticket ordering untested
**Location:** internal/parentlock/store.go:240
**Issue:** The sort tie-break among front tickets and the rule that a front acquirer waits behind an older front ticket have no row.

### [LOW] Watch-open failure fallback untested
**Location:** internal/parentlock/acquire.go:115
**Issue:** The Warn branch and the poll-only path cannot be reached from a test because fswatch.Watch is not injectable.

### [NIT] Stale queue-head wording on Store.Covered
**Location:** internal/parentlock/store.go:107
**Issue:** The field comment says "oldest live ticket", while the queue now orders front tickets first.

## Rejected

- Verify skipped on a differing tree (focus bullet 2): `gateAndFix` runs `gate.check` on every pass, and the skip applies only to a tree already recorded as passed, so a tree that differs from the verified one cannot land unverified; the summary and doc.go claims hold.
- Lost wake-up between the last attempt and the watch start: the first poll delay is the one-second floor, and later events are delivered by the unbuffered forward channel when `pause` selects, so no wake is lost beyond the floor.
- Leaked watcher on return: the single `defer watcher.Close()` in `Acquire` runs on every return, and `pause` always drains its sleep goroutine.
- Stale or abandoned front ticket: a cancelled, paused or failed wait removes it through `abandonTicket`, a grant removes it in `tryAcquire`, and a dead holder's ticket is pruned by `liveTickets`.
- Lock re-check tests moved to the parent-records commit: the release-before-landing and retry cases still reach `recheckLock`, and the old release-during-fixer case cannot occur now that the fixer runs outside the lock.
- Frees the lock for fix review sessions and moves the Baseline, neither named by the entry: both follow from the entry's design (a fix review is an agent session on the task worktree alone, and the freed lock makes the parent move during the fix); the summary names both and the docs say what the code does, so no scope finding.
- docs/overview.md, the package docs and pattern/PATTERN-daemon-wakeups.md: each says what the code now does; the directive declares no PATTERN change and no commit carries a Pattern-Departure trailer.
