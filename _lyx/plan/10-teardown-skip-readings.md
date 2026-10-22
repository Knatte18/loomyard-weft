# Card 10 — Teardown skips the driver checks for a run with nothing left to do, and waits for the run lock before ending it

**Create:**
- `plan:internal/pairteardown#Teardown.driverChecksSkipped` -> `func (t *Teardown) driverChecksSkipped(req Request) (bool, error)`
- `plan:internal/pairteardown#Teardown.pausedIn` -> `func (t *Teardown) pausedIn(slug string) (bool, error)`
- `plan:internal/pairteardown#Teardown.awaitRunLockOrPass` -> `func (t *Teardown) awaitRunLockOrPass(ctx context.Context, slug string, paused bool) error`
- `plan:internal/pairteardown#runLockHolderPasses` -> `func runLockHolderPasses(holder int, holderKnown bool, waiting []pauserecord.WaitingMarker, registered []runproc.Record, slug string, paused, parentLocked bool) bool`
- `plan:internal/pairteardown#TestRunLockHolderPasses` -> `func TestRunLockHolderPasses(t *testing.T)`

**Edit:**
- `internal/pairteardown#Request`
- `internal/pairteardown#Teardown`
- `internal/pairteardown#New`
- `internal/pairteardown#Teardown.EndSession`
- `internal/pairteardown#Teardown.busyError`
- `internal/pairteardown#Teardown.teardownEnded`
- `internal/pairteardown#fakeSubstrates`
- `internal/pairteardown#fakeSubstrates.teardown`
- `internal/pairteardown#TestTeardown_CallSequence`
- `internal/pairteardown#TestTeardown_OriginSeams`
- `internal/pairteardown/doc.go`
- `pattern/PATTERN-pair-teardown.md`
- `docs/overview.md`

**Uses:**
- `plan:internal/pauserecord#ListWaiting`
- `plan:internal/pauserecord#WaitingMarker`
- `plan:internal/runproc#List`
- `plan:internal/runproc#Record`
- `plan:internal/pairteardown#Teardown.registeredBattenAlive`
- `internal/pairteardown#awaitRunLockFree`
- `internal/pairteardown#Teardown.childLanded`
- `internal/pairteardown#Teardown.readBoardClaim`
- `internal/parentlock#Store.HeldBy`

**Intent:**
D5's readings inside `pairteardown`, so `fabric remove` (card 11), `reed down` (card 12) and the reconcile's `Teardown` seam take one rule.

`Request` gains `Verdict fabricengine.EntryState`, the caller's origin reconcile verdict for the slug, empty when it ran none.
`driverChecksSkipped` is read only while the task worktree is present, since a gone one already skips the quiet wait and both run-lock waits and its status cannot be read (D4).
It reads true, in this order and stopping at the first that holds:
the verdict is `ended` or `owned_elsewhere`;
a live pause entry covers the slug (`pausedIn`, a hub or a slug entry of the hub's pause record, read as `HubSeamOptions`' `HubPaused` reads it);
the local board entry reads `done` (`readClaim`), as D4 reads it;
the task worktree's status history holds a Finalize Done (`landed`) and batten is not running, which is no live registered batten (`battenAlive`, card 9), since batten still runs Land and deploy after Finalize Done.
These are the PATTERN entry's "paused, ended, owned by another machine, or done locally with batten no longer running".
A read error is returned, with nothing ended.

With the checks skipped, `EndSession` runs D4's other order: the run-lock wait, the refusal probe, `End`, the driver-lock re-check, the loop end, the loop end's own run-lock wait, the session end; there is no quiet wait.
The wait is a new substrate, `awaitOrPass func(ctx context.Context, slug string, paused bool) error`, wired in `New` to `awaitRunLockOrPass` and faked in `fakeSubstrates.teardown`, so the sequence rows script its answer.
`awaitRunLockOrPass` waits, bounded by `runLockWait` through `awaitRunLockFree`, so a step doing local work under a pause, such as a merge under the parent lock, finishes before `End` ends the loop and the step with it.
At the bound it reads the newest unfinished step's pid (`shedverbs.RunningStepPID`), the live waiting markers (`pauserecord.ListWaiting`, with `parentlock.ProcessAlive`), the slug's live owner records and whether a parent lock record names the slug, and `runLockHolderPasses` decides.
The holder passes, and is ended by `End` and the loop end, in two cases: a live waiting marker of the slug names that step's pid or the pid of a process registered under the slug (a holder waiting at the network gate, the incident's case); or a live pause entry covers the slug and no parent lock record names it (a step waiting on its interrupted agent, which the loop's watch never kills under a pause).
Any other holder refuses with `ErrDriverBusy`, with nothing ended: a known holder names its pid and the way forward of retrying the removal once that pid ends; an unknown holder names the run lock and the way forward of retrying the removal once `lyx tender status` lists no owned process of the slug.

Without the skip the quiet wait and its refusal stand.
`busyError` keeps its blocker and drops "end it with `lyx reed down` in the pair", since card 12's `reed down` refuses exactly this running, unpaused run; its way forward becomes retrying once the driver has written its report, or `lyx tender pause <slug>` and then the removal again, which the skip then lets through, and it adds the slug's live owned processes (role and pid).
`teardownEnded` passes `Verdict: ended`, so the reconcile's local removal of an ended entry takes the skip.

`TestRunLockHolderPasses` is a table over the pure rule: marker naming the step pid passes; marker naming a registered pid of the slug passes; marker of another slug does not; paused with no parent lock passes; paused with a parent lock record refuses; unpaused with no marker refuses; unknown holder with a matching registered marker passes; unknown holder with no marker refuses.
`TestTeardown_CallSequence` gains fake `paused`, `awaitOrPass` and `driverChecksSkipped` read answers and rows: an `ended` verdict, an `owned_elsewhere` verdict, a slug pause and a hub pause each call `awaitOrPass, refusal, endOwned, driverLock, kill, seize, await, end` with no `quiet`; a board entry done skips whatever batten reads; a Finalize Done with no live registered batten skips, and with a live one runs the quiet wait; an `awaitOrPass` that passes goes on to `endOwned`, and one that refuses with `ErrDriverBusy` returns it and calls no `endOwned`; a skipped-order refusal probe that refuses calls no `endOwned`; a skip-reading error is returned with no `endOwned`; a gone task worktree makes no skip reading.
Its existing busy-driver row, which asserts "lyx reed down" (`internal/pairteardown/teardown_test.go:189`), asserts `lyx tender pause slug` instead, and the owned processes named.
`TestTeardown_OriginSeams` gains a row: the `Teardown` seam's local removal carries `Verdict: ended`, so a busy driver under the fake substrates is skipped past and the removal runs.

Docs.
`doc.go` gains the skip readings, the pre-`End` run-lock wait and its two pass cases.
`pattern/PATTERN-pair-teardown.md`'s opening line becomes the declared text: "Every teardown of a pair goes through `internal/pairteardown`, which ends the pair's owned processes and then its reed session before any worktree is removed; its driver checks are skipped for a pair that is paused, ended, owned by another machine, or done locally with batten no longer running, never its run-lock wait while the task worktree is present, which then runs before the owned processes are ended."
Its quiet-wait bullet names the skip and the pre-`End` run-lock wait.
`docs/overview.md`'s `internal/pairteardown/` module row reads: quiet wait or the skip's run-lock wait, refusal probe, owned-process end, loop and reed session end, then fabric removal.

**ImpactSummary:** Request gains a field every existing caller leaves empty; EndSession's skipped path and busyError's way forward change inside pairteardown only, and the one test asserting busyError's wording is updated here.

**Commit:** 10: teardown skips the driver checks of a paused, ended, foreign or locally done run
