---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/09-land-row.md (Tests, TestNew_PauseDuringLandReentersTheRowThroughGotoThenRun)
    summary: the planned pause test sets pause_requested on the status before the step, so shed's step 3 records the run paused and never calls Land, and the test cannot reach the Land pause Stuck it asserts
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/01-gitrepo-push-retry.md (Tests)
    summary: no planned test reaches the ordering card 1 calls required, an exhausted retryable refusal returned by pushWithRebaseRetry and PushRebaseFree before rebaseRetryTriggers, so a rebase or ErrPushRejected on that path goes uncaught
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/11-orch-docs.md (orch-template-role.md bullets)
    summary: card 11 leaves run-loop item 4 and Investigating a stop sending every [batten notice] to Investigating a stop, which contradicts its new After landing text that a ready-task notice is the cue to start tasks
  - id: F4
    severity: LOW
    class: scope
    location: _lyx/plan/08-land-producer.md (Intent step 1 and step 7, Tests)
    summary: two Land failure branches have no named test, the second ReadyTasks read in newlyReady after the prune and a PauseRequested read error, and card 8 never says how the latter is handled
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/plan/07-row-failure-notice.md (Tests)
    summary: the BattenHistoryLen-error branch, which warns and treats the failure as a new episode, has no named test
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/plan/11-orch-docs.md (internal/orchcli/doc.go paragraph)
    summary: the planned orchcli doc.go run loop omits D8's hand deploy while deploy is empty, and step 4 on blocked runs does not mention batten's automatic first Publish verify retry
  - id: F7
    severity: LOW
    class: scope
    location: _lyx/plan/11-orch-docs.md (orch-template-role.md bullets)
    summary: the orch stencil's Acting without asking section still lists land and clean up among the orch's own actions, and card 11 leaves it as it is
  - id: F8
    severity: NIT
    class: consistency
    location: _lyx/plan/11-orch-docs.md (wordingClaims paragraph)
    summary: card 11 drops two After landing claims on closing a batten run by hand, but claims_test.go has only one such claim
  - id: F9
    severity: NIT
    class: scope
    location: _lyx/plan/09-land-row.md (Create landEntry)
    summary: card 9 does not say which file holds landEntry, and the entries_batten.go header that counts four batten registry entries goes stale if it lands there
---

### [MEDIUM] The planned Land pause test cannot reach Land

**Location:** `_lyx/plan/09-land-row.md`, Tests, `TestNew_PauseDuringLandReentersTheRowThroughGotoThenRun`.

**Issue:**
The test is specified as stepping the shed "with batten seeded at Land, `pause_requested` set on its status, the land lock contended and `Land.PauseRequested` reporting the pause", and expects "the step halts blocked at Land with the flag still set".
`internal/shedengine/run.go:215` checks `st.PauseRequested` in step 3, before calling the producer.
When the flag is set, that branch persists `StatePaused` with `consumePause` true and returns, so `Land.Call` never runs.
The step therefore ends paused with the flag cleared, not blocked at Land with the flag set.
The planned setup cannot reach the pause Stuck that Shared Decision `land-row-pause` prescribes.
Without a test that reaches it, the decision's chain (blocked persist leaves the flag set, then goto clears it, then run re-enters Land) is untested end to end through shed.
The Shared Decision's own premise holds: the blocked branch at `run.go:330-335` calls `persist(..., false, "")` and leaves the flag set, and `goto.go:95` clears it.

**Fix:**
In the test, set `pause_requested` on batten's status from inside a fake that runs during `Land.Call`, for example the contended `Land.LandLock.Acquire` or `Land.Sleep`, and have `Land.PauseRequested` read it.
Step 3 then passes, Land sees the pause mid-wait, and the step halts blocked at Land with the flag still set.
Keep the rest of the sequence as planned: goto onto Land clears the flag, and the next step calls Land again.

### [MEDIUM] The push retry's ordering before the rebase triggers is untested

**Location:** `_lyx/plan/01-gitrepo-push-retry.md`, Tests.

**Issue:**
Card 1 says an exhausted error is returned "before `rebaseRetryTriggers` is consulted, so a retryable refusal never triggers `pull --rebase`; this ordering is required because the substring `rejected` in `rebaseRetryTriggers` matches every `! [remote rejected]` line".
It also says `PushRebaseFree` returns such an error "never mapped to `ErrPushRejected`".
Today `internal/gitrepo/push.go:78` and `:127` map any stderr holding `rejected` to the rebase path or `ErrPushRejected`.
The planned tests cover only `retryablePushRefusal` and `PushWithRetry` with a fake push.
No test drives `Repo.Push`, `PushCoalesced` or `PushRebaseFree` into a retryable refusal.
If the rewired primitives check `rebaseRetryTriggers` first, or wrap the loop in the wrong place, every planned test still passes.
That would break D6 and the acceptance criterion "with no rebase and no `ErrPushRejected` between attempts", and card 2's coalesced Warn would take the divergence branch again.
The criterion "a local `cannot lock ref` fails at once" is pinned only at the classifier, not at a primitive.
A deterministic remote `cannot lock ref` exists: pushing a branch `x/y` to a remote that holds a branch `x`.
`internal/gitrepo/push_test.go` (integration tier) already has a bare-remote fixture.
The obstacle is the production backoff: `PushWithRetry`'s nil sleep sleeps for real, and `Repo` has no sleep seam, so a test would wait out the production delays, which `PATTERN-test-speed` forbids.

**Fix:**
Give the three primitives an injectable sleep, for example an unexported package-level sleep variable or a `Repo` field that a test in package `gitrepo` overrides.
Then add rows to the existing integration table in `internal/gitrepo/push_test.go`, with no new top-level test.
Each row pushes an `x/y` branch against a remote holding `x`.
`PushRebaseFree` must return an error satisfying `errors.Is(err, ErrPushRetriesExhausted)` and not `ErrPushRejected`.
`Push` must return the same, and the clone must have run no `pull --rebase`: HEAD unchanged and no rebase in progress.
Both pushes must be made `pushMaxAttempts` times, counted through the injected sleep.
List the seam as a target of card 1.

### [MEDIUM] Ready-task notices are still routed to Investigating a stop

**Location:** `_lyx/plan/11-orch-docs.md`, `contracts/stencils/orch/orch-template-role.md` bullets.

**Issue:**
D4 and D8 add a second kind of `[batten notice]`: the ready notice, which is "the cue to start the named tasks" and not a stop.
Card 11 says this in `## After landing`.
The stencil's `## The run loop` item 4 still says "A line starting with `[batten notice]` is the wake-up: handle it as under Investigating a stop".
`## Investigating a stop` still opens "A stop is a `[batten notice]`, a halted run, …".
Card 11 edits neither.
The stencil then gives the orch two answers for a ready notice.
Under item 4 the orch may read `lyx batten status` for a run that is done and has nothing to investigate, or fork for it.
That is the false wake D8 is meant to remove.

**Fix:**
Add to card 11's stencil bullets:
run-loop item 4 says a `[batten notice]` naming newly ready tasks goes to `## After landing`, and every other notice to Investigating a stop;
`## Investigating a stop`'s first sentence excludes the ready notice, as it already excludes a parent-review request.
Add a claim to `wordingClaims` in the run-loop or Investigating-a-stop section pinning the exclusion.

### [LOW] Two Land failure branches have no named test

**Location:** `_lyx/plan/08-land-producer.md`, Intent steps 1 and 7, Tests.

**Issue:**
`ReadyTasks` is read twice: in step 5 for the snapshot, and in step 7 by `newlyReady`, after `PruneBoard` has run.
The failing-step table names "a board read" once, so the step-7 read error, a Stuck after the prune, is not named.
`LandDeps.PauseRequested` returns `(bool, error)`, and the land-lock and dirty-prime waits poll it.
Card 8 never says what a read error there does.
Shared Decision `land-steps-stuck` implies a Stuck, but neither the branch nor a test is named.

**Fix:**
Name the step-7 `ReadyTasks` error as its own row of the failing-step table, asserting the Stuck and that a re-entry with the snapshot present then sends the ready notice.
State in Intent step 1 that a `PauseRequested` read error in either wait is Stuck naming the step, and add a row for it.

### [LOW] Row-notice episode fallback is untested

**Location:** `_lyx/plan/07-row-failure-notice.md`, Tests.

**Issue:**
Card 7 prescribes "A `BattenHistoryLen` error is warned about and reads as a new episode, since a repeated notice costs less than a lost one".
`TestWorktreeTeardown_FailureNoticeOncePerEpisode` and `rownotice_test.go` cover the same-length, grown-length and cancelled cases, the missing strand, the spacing and the drop.
No row makes `BattenHistoryLen` return an error.

**Fix:**
Add a row to `TestWorktreeTeardown_FailureNoticeOncePerEpisode`, with no new test: a `BattenHistoryLen` error after a queued notice still queues a notice.

### [LOW] orchcli doc's run loop drifts from D8 and the automatic retry

**Location:** `_lyx/plan/11-orch-docs.md`, the `internal/orchcli/doc.go` paragraph.

**Issue:**
Card 11's step 5 says "batten's Land row lands the run and the orch acts only on a Land or teardown failure notice or a ready-task notice".
D8 also keeps the orch's hand deploy "while the hub's `deploy` key is empty", which the stencil bullet keeps, so "only" disagrees with the stencil.
Step 4 (`internal/orchcli/doc.go:34`) tells the orch to rerun failing tests on any blocked run.
It is left unchanged, though batten now retries the first Publish verify failure before any notice.

**Fix:**
In card 11, add the hand deploy while `deploy` is empty to step 5.
Have step 4 say a blocked Publish verify failure reaches the orch only on its second occurrence.

### [LOW] Acting without asking still has the orch land

**Location:** `_lyx/plan/11-orch-docs.md`, `contracts/stencils/orch/orch-template-role.md` bullets.

**Issue:**
`## Acting without asking` reads "start the runs the operator approved, un-wedge stuck ones, approve, land and clean up".
After D1 and D8, landing and cleanup are batten's.
Card 11 rewrites `## After landing` but leaves this list as it is.

**Fix:**
Add a bullet to card 11 that drops "land and clean up" from that list, or rewords it to handling a Land or teardown failure.

### [NIT] Claim count on After landing

**Location:** `_lyx/plan/11-orch-docs.md`, `wordingClaims` paragraph.

**Issue:**
Card 11 says the row "drops the two `## After landing` claims on closing a batten run by hand".
`contracts/stencils/claims_test.go:669` holds the only such claim.
The other `## After landing` claim, line 668, is the drive-report one, and card 11 keeps it.

**Fix:**
Say "the `## After landing` claim on closing a batten run by hand".

### [NIT] landEntry's file is unnamed

**Location:** `_lyx/plan/09-land-row.md`, Create `plan:internal/shedrecipe#landEntry`.

**Issue:**
`internal/shedrecipe/entries_batten.go`'s header says it "implements the four batten registry entries" and names them.
landEntry shares their `Env.Slug`/`ScratchDir`/`PrimeLock` validation shape, so it belongs in that file.
Card 9 does not say which file holds it, and that header goes stale if it lands there.

**Fix:**
Name `entries_batten.go` as landEntry's home and have its header stop counting entries.

## Focus coverage

- Focus 1 (row-count and registry tests):
  every unit-tier pin is a target.
  `TestNew_RowNamesMatchTheDurableIdentityConstants`, `TestNameRunShed_ValueIsPinnedAgainstASymmetryRename` and the four-engine test are covered by edit or delete.
  `TestNames`/`ShipsExpectedEntries`, `minimalEntries`, `battenRowEngines` and `lifecycleEntryCases` are edited.
  `TestBuild_EveryRegisteredEngineBuilds` (`internal/shedbuild`) ranges over `Names()` on `envkit.FullEnv` with a nil config, which card 9's `FullEnv` edit satisfies.
  In the integration tier, `fourRowRunForRecipe`, `stepAwaitingApprovalResumeDoneTeardown_ArchivesTheRunRecords` and `shortPollBattenRecipe` are targets.
  `stepRunShedPausedChild_WaitsThenTearsDownOnceDone` is not a target, but it steps until the pair is gone, up to five steps.
  It now passes through a real-wired Land on the same hubforge hub that card 9 asserts Land returns Done over, and its comment that the run "reaches Worktree-Teardown" stays true.
  Not a defect.
- Focus 2 (pause during Land):
  shed clears `PauseRequested` only in a paused persist or a goto (`run.go:210,219,302`, `goto.go:95`), and the blocked Stuck at `run.go:332` leaves it set, so the Shared Decision is right.
  `gotoAdmitted` admits the current row for a blocked run, and card 9's `preGoto` row admits Land with batten at Land.
  The planned shed-level test does not reach the branch: F1.
- Focus 3 (Stuck and refusal reachability):
  card 8's empty parent, its seam read errors and the cards-9/10 branches each have a named input and test, except the two in F4.
  That includes card 10's driver-busy resume refusal and `ErrChildRunBusy` via `TestChildSpawnError`.
- Focus 4 (cards 1 and 2 push tests):
  card 2's two remaining rows pin what only the wrapper does, the argv and dir and that the seam's `sleep` and `branch` reach `PushWithRetry`, while card 1 owns the loop and the classifier.
  Not a defect.
  The gap is at the primitives instead: F2.
- Focus 5 (card 11): F3, F6, F7, F8.
