---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/fabricengine/bolt.go:301
    summary: The pause gate is passed after an outer shared lock is already held, so a pause still holds a waiter that keeps the board write lock or the pair's teardown lock.
    fix: In internal/fabricengine/bolt.go, move the pauserecord.Admit(pushLockGateOp) call out of pullFirst and into Bolt.Pull, after its SkipGit return and before b.acquireWriteLock, and into Bolt.PullThenCommitWritten before b.acquireWriteLock, each released by a defer after the write lock's release. In internal/fabricengine/pushanchored.go PushPairAnchored, call pauserecord.Admit(pushLockGateOp) after the SkipGit/SkipPush return and before acquireTeardownShared, deferring its release, so the teardown lock is never held at the gate; the inner Admit in coalesceFlagged then nests and passes at once. Update the pullFirst doc, the PushPairAnchored doc and the doc.go paragraph at 1241 to say the gate is passed before the outermost lock, and delete the summary's design-decision bullet that keeps the teardown lock before the gate. Add a row to TestCoalescePushBothAt_HeldByAPauseWithoutThePushLock or a sibling Bolt test that probes the board write lock from the gate's Sleep while a pause holds Bolt.Pull.
    origin: lens:code-correctness
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lym/landing/summary.md#Design decisions
    summary: Goal point Dobby starts no batten driver for a paused pair is built by no change, and the summary's reason rests on a false premise, since Pause writes its live entry before it lists sessions.
    fix: Find the path that started the batten run for the covered pair in the 2026-11-18 incident from the dobby and batten logs; dobby reads coverage through beamcli/dobbydeps.go PauseCovers, planPass and readyToStart. If a spawn path ignores the pause, gate it on PauseCovers and add a row to the dobbyengine test of that path. If none exists, rewrite the summary bullet to drop the claim that the pause failed on the missing reed session, cite TestPlanPass's VerdictPauseCovers rows and the starts_test.go row APauseCoveringTheFrontTaskHoldsItAndTheOthersBehindIt as the Done-when test, and say the incident's spawn path was not found.
    origin: lens:code-scope
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/fabricengine/pull.go:503
    summary: Four of the six new gate sites, lockWeftForReplay, Bolt.pullFirst, retryBoltPush and takeoverBranch.land, have no test that fails if their Admit is removed or moved after the lock.
    fix: For each of lockWeftForReplay (pull.go), Bolt.Pull or its new gate site after F1, retryBoltPush (pushclassified.go) and takeoverBranch.land (takeoverwrite.go), add a row to the existing test of the public entry point that reaches it, installing a gate whose Sleep asserts the site's lock is free from a fresh handle and then lifts the pause, as TestCoalescePushBothAt_HeldByAPauseWithoutThePushLock does; assert one wait and the operation's normal result.
    origin: lens:code-tests
  - id: F4
    severity: MEDIUM
    class: consistency
    location: internal/fabricengine/coalesce_test.go:121
    summary: The CodePushed and BothPushed rows that asserted a pushed side's flag stays off are removed with no Loosened-Check trailer, and coalesceFlagged's re-acquire pass now puts such flags back.
    fix: Add a commit to the range whose message carries a Loosened-Check trailer naming TestPushDeferral_PutsBackTheUnpushedFlags rows CodePushed and BothPushed and the cause, that the deferral now fires only at a gate before the lock, with the re-acquire case F5 describes stated as accepted.
    origin: assist:assist
  - id: F5
    severity: LOW
    class: consistency
    location: internal/fabricengine/pendingflags.go:61
    summary: The pushDeferral doc and doc.go say the deferral fires when no side has been pushed, but coalesceFlagged admits on every loop pass, so after a pushed step it can fire and put back flags of pushed sides.
    fix: In the pushDeferral doc comment and the doc.go lines at 1305, state that the deferral fires at the gate before each acquire, on the first acquire with nothing pushed, and on a later acquire after earlier steps pushed, when the flags it puts back may name sides with nothing left to push; change the Info log text so it does not claim the commits stay local.
    origin: lens:code-correctness
  - id: F6
    severity: LOW
    class: design
    location: internal/fabricengine/pendingflags.go:155
    summary: No test shows that coalesceFlagged releases its admission when the acquire misses or errors; a leak would let every later network call of the process bypass the pause.
    fix: Add a row to TestCoalescePushRequest_CoveredCallDefers or a sibling that holds the push lock from another handle, runs CoalescePushRequest with an uncovered gate installed, and then asserts that a pauserecord.Network call under a covering entry waits once.
    origin: lens:code-tests
  - id: F7
    severity: LOW
    class: design
    location: internal/tenderengine/resume.go:479
    summary: Revive's new skip of a pair with no reed session has no test.
    fix: Add a row to TestRevive in internal/tenderengine/resume_test.go with one pair whose sessionsErr is errNoReedSession and one stalled pair; assert the first is in Skipped with the noReedSessionNote reason and the second is revived.
    origin: lens:code-tests
  - id: F8
    severity: LOW
    class: design
    location: internal/tenderengine/pause.go:243
    summary: No row shows that a Sessions error other than reedengine.ErrNoSession still fails pause and status, so a match widened to every error passes the suite.
    fix: Add a row to TestPause and a subtest to TestStatus whose pair sets sessionsErr to a plain errors.New value, asserting the call returns that error and lists the pair under no skipped.
    origin: lens:code-tests
  - id: F9
    severity: LOW
    class: consistency
    location: internal/pauserecord/doc.go:61
    summary: The package doc names remote-refs as the operation that marks a lock hold; the new push-lock operation marks one too.
    fix: In internal/pauserecord/doc.go, change the sentence to say an operation an Admit caller names, such as remote-refs or push-lock, marks a lock hold.
    origin: lens:code-affected-set
  - id: F10
    severity: LOW
    class: consistency
    location: docs/overview.md:874
    summary: The tender overview lists every part of lym tender status but not the new skipped part, and its pause text does not say a pair with no reed session is skipped.
    fix: In docs/overview.md, add the skipped part to the status sentence at 874, and add to the pause sentence at 868 that a pair with no reed session is reported under skipped and the pause goes on.
    origin: lens:code-affected-set
  - id: F11
    severity: LOW
    class: consistency
    location: internal/tenderengine/doc.go#Revive
    summary: The tenderengine Revive section and the revive help text in tendercli/cli.go do not say a pair with no reed session is reported under skipped.
    fix: Add one sentence to the Revive section of internal/tenderengine/doc.go and to the revive Long text in internal/tendercli/cli.go saying a pair with no reed session is reported under skipped and the revive goes on.
    origin: lens:code-affected-set
  - id: F12
    severity: NIT
    class: consistency
    location: internal/tendercli/cli.go:343
    summary: The status help says skipped lists each pair with no reed session, but the prime is listed there too.
    fix: Change the line to say skipped lists each pair, the prime included, with no reed session.
    origin: lens:code-affected-set
---

### [BLOCKING] The gate is passed inside an outer shared lock

**Location:** `internal/fabricengine/bolt.go:301`, `internal/fabricengine/bolt.go:215`, `internal/fabricengine/bolt.go:273`, `internal/fabricengine/pushanchored.go:173`

**Issue:** The goal says a pause hold never waits while holding a shared lock.
`Bolt.Pull` and `Bolt.PullThenCommitWritten` take the board write lock and then call `pullFirst`, whose new `Admit` waits at the gate with that lock held.
Every other board writer then blocks on the write lock until the pause lifts.
`PushPairAnchored` likewise takes the pair's teardown lock before `coalesceFlagged` reaches its `Admit`.
A held push then blocks that pair's teardown and handover, and a takeover of any run whose parent is that pair, since `TakeOverRun` takes the parent's teardown lock too.
The summary and `doc.go:1245` say the gate is passed before the first lock, which these two paths contradict.
The summary records the teardown order as a design decision, but the board entry makes no exception for it.

### [MEDIUM] Dobby's goal point rests on a false premise

**Location:** `_lym/landing/summary.md#Design decisions`

**Issue:** The diff changes no dobby code.
The summary says no change is needed because the pause in the incident was failing on the missing reed session.
`Pause` writes its live entry through `pauserecord.Update` before `coveredSessions` lists the sessions (`internal/tenderengine/pause.go`), both before and after this diff.
A pause that failed at the listing therefore still left the pair covered, and dobby's `PauseCovers` reads that record.
The incident's driver start is unexplained.
Existing tests pin that `planPass` gives no pass under a covering verdict, and that `readyToStart` refuses a covered task.
Neither explains how pid 15436 started.

### [MEDIUM] Four gate sites are untested

**Location:** `internal/fabricengine/pull.go:503`, `internal/fabricengine/bolt.go:301`, `internal/fabricengine/pushclassified.go:811`, `internal/fabricengine/takeoverwrite.go:173`

**Issue:** Only the `coalesceFlagged` and `coalescePush` sites are pinned, by `TestCoalescePushBothAt_HeldByAPauseWithoutThePushLock` and the board sync row.
Removing the `Admit` call from any of the other four sites, or moving it after the lock, passes the suite.

### [MEDIUM] Removed flag assertions carry no Loosened-Check trailer

**Location:** `internal/fabricengine/coalesce_test.go:121`

**Issue:** The base table asserted that a pushed side's flag stays off after a deferral.
The diff drops those rows, and no commit in the range carries a `Loosened-Check:` trailer.
The dropped check is not moot: on `coalesceFlagged`'s re-acquire pass a deferral now puts back flags of sides already pushed.

### [LOW] The deferral doc overstates when it fires

**Location:** `internal/fabricengine/pendingflags.go:61`, `internal/fabricengine/doc.go:1305`

**Issue:** `coalesceFlagged` calls `Admit` before every acquire, so a deferring child covered after its first step fires the deferral with sides already pushed.
The doc says no side has been pushed then.
The log line says the commits stay local, which is false on that path.

### [LOW] Admission release on a missed acquire is untested

**Location:** `internal/fabricengine/pendingflags.go:155`

**Issue:** `admissions` is process-wide (`internal/pauserecord/hold.go`).
A leaked admission on the miss or error branch would let every later network call pass a covering pause, and no test reaches that branch with a gate installed.

### [LOW] Revive's skip is untested

**Location:** `internal/tenderengine/resume.go:479`

**Issue:** `TestRevive` has no row that sets `sessionsErr`, so the new skip under `ReviveReport.Skipped` is unpinned.

### [LOW] The narrow ErrNoSession match is unpinned

**Location:** `internal/tenderengine/pause.go:243`, `internal/tenderengine/resume.go:691`

**Issue:** `fakePair.sessionsErr` is only ever set to `errNoReedSession`, so no row fails if the skip widens to every `Sessions` error.

### [LOW] pauserecord doc names one lock-hold operation

**Location:** `internal/pauserecord/doc.go:61`

**Issue:** The sentence reads as if `remote-refs` were the only operation marking a lock hold, but `fabricengine` now marks `push-lock` through `Admit` as well.

### [LOW] The overview omits the skipped part

**Location:** `docs/overview.md:874`, `docs/overview.md:868`

**Issue:** The overview enumerates every part of `lym tender status`, and the new `skipped` part is missing.
The pause description does not mention the no-session skip.

### [LOW] Revive docs omit the skip

**Location:** `internal/tenderengine/doc.go#Revive`, `internal/tendercli/cli.go:278`

**Issue:** The diff documents the no-session skip for pause and status but, for revive, only on the `ReviveReport.Skipped` field.

### [NIT] Status help leaves out the prime

**Location:** `internal/tendercli/cli.go:343`

**Issue:** `sessionStatuses` reports the prime under `skipped` too, as `StatusReport.Skipped`'s comment says.

## Rejected

- Assistant F2 (missing sessions disappear when `shuttlePair.Sessions` returns early on no running records): a pair with no running records has no session and no running turn, so listing nothing is right and matches the base behaviour; the goal concerns the refusal, not this case.
- Assistant F3 (the lock test runs no real batten driver): `pushLockFree` takes the same `fabric.push.lock` path from a fresh handle, which is what another pair's push does; the test asserts the outcome the Done-when names.
- Assistant F4 (revive skip is unrequested scope): `Revive` shares `coveredSessions`, and the skip is the minimal handling that change forces; a revive cannot act on a pair with no session either way.
- The assistant's merged F1 is kept as F1; its F6 is kept as F3.
