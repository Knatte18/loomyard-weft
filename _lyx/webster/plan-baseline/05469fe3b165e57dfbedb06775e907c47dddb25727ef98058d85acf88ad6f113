# Card 30 — finalize-recheck-and-land

**Create:**
- `plan:internal/landingshed#parentGuard.stillHeld` -> `func (g parentGuard) stillHeld(producer string) (string, error)`
- `plan:internal/landingshed#taskTrailer` -> `func taskTrailer(block, slug string) string`
- `plan:internal/landingshed#TestFinalize_LockRecheckAndLanding` -> `func TestFinalize_LockRecheckAndLanding(t *testing.T)`

**Delete:**
- `internal/landingshed#Finalize.closePullRequest`
- `internal/landingshed#TestFinalize_ClosingPullRequestAfterLanding`

**Edit:**
- `internal/landingshed#Finalize.Call`
- `internal/landingshed#Finalize.pushParent`
- `internal/landingshed#TestFinalize_MergeOptionsCarriesComposedMessage`
- `internal/landingshed#TestFinalize_MergeMarkPushOrderAndVerdict`
- `internal/landingshed_test#TestFinalize_OverRealHub`

**Uses:**
- `internal/landingshed#fakeFixerSeats`
- `internal/landingshed#LandingStopPrefix`

**Extends:**
- `internal/landingshed/doc.go`

**Intent:** Finalize re-checks its lock before landing and lands with no pull request (D10 steps 5 and 6, D13).
Immediately before the landing, `parentGuard.stillHeld` re-reads the parent lock record through the guard's store: a record that names this run passes; a record that no longer names it means something released it, an owner pause, another machine's ownership or an operator's `lyx batten lock release`, so Finalize lands nothing and takes no lock, and is Stuck with Halt naming the release and its three possible causes, with the way forward `lyx loom resume` once no owner pause stands and the run is this machine's, which after an operator's release holds already.
An unreadable record is a Stuck with Halt, as at the guard's acquire.
Both halts open with `LandingStopPrefix`, since they are D10 halts only the orch's `lyx loom resume` may lift: the hand-back predicate then keeps the tackler, the arbiter and batten's re-step from resuming them before the cause clears.
The re-check runs immediately before every landing attempt, the retry's landing after a merge-in-required error included, so a release during the retry's fixer wait is caught the same way.
The resume re-runs Finalize from step 1 under a lock taken afresh, so a resume made before the cause clears lands nothing: a run another machine owns stops at the owner check, and a lock the owner-pause arm releases again is caught by this same re-check.
The re-read and the landing are not atomic: a release between them is caught, as today, only by the parent push's rejection.
The landing commits the parent's run records, squashes with Describe's text and the trailer block as the commit message, the block ending with a `Lyx-Task: <slug>` line `taskTrailer` appends, marks the board task done and pushes the parent; no pull request is opened, looked up or closed, so `closePullRequest` goes and Finalize makes no GitHub call.
`pushParent` keeps its transient re-step, its repair note and its config-change notice, and the deferred release frees the lock after it.

`TestFinalize_LockRecheckAndLanding` drives `Call` over a real `parentlock.Store` in a temporary directory and `fakeFixerSeats`, whose turn callback releases the run's record through that store, so the release happens between the guard's acquire and the re-check within one `Call`.
It pins: a record released while the fixer waited halts with `LandingStopPrefix` before landing, with no squash and no push, and its resume re-runs from step 1 under a fresh lock; one whose run is then owned by another machine stops at the owner check without landing; one an operator's release removed halts the same way and lands on the resume alone; a release during the retry after a merge-in-required error halts before the retry's landing; an unreadable record halts.
The message and order tests, which this card edits, assert the `Lyx-Task: <slug>` trailer line and that no GitHub call is made, and the real-hub test asserts the landed commit carries the trailer.

`doc.go`'s Finalize section describes the re-check, its non-atomic bound and the landing without a pull request.

**ImpactSummary:** Finalize gains its pre-landing lock re-check and the task trailer and loses its pull-request close, the one GitHub call left in the package's landing path.
