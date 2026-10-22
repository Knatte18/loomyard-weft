# Card 10 — Teardown skips the driver checks for a run with nothing left to do, and waits for the run lock before ending it

**Create:**
- `plan:internal/pairteardown#Teardown.driverChecksSkipped` -> `func (t *Teardown) driverChecksSkipped(req Request) (bool, error)`
- `plan:internal/pairteardown#Teardown.pausedIn` -> `func (t *Teardown) pausedIn(slug string) (covered, hubEntry bool, err error)`
- `plan:internal/pairteardown#Teardown.awaitRunLockOrPass` -> `func (t *Teardown) awaitRunLockOrPass(ctx context.Context, slug string, paused, hubEntry bool, retry string) error`
- `plan:internal/pairteardown#runLockHolderPasses` -> `func runLockHolderPasses(holder int, holderKnown bool, waiting []pauserecord.WaitingMarker, slug string, paused, parentLocked bool) bool`
- `plan:internal/pairteardown#runLockRefusal` -> `func runLockRefusal(holder int, holderKnown bool, slug, lockPath, goos, retry string, paused, hubEntry, parentLocked bool) error`
- `plan:internal/pairteardown#TestRunLockHolderPasses` -> `func TestRunLockHolderPasses(t *testing.T)`
- `plan:internal/pairteardown#TestAwaitRunLockOrPass` -> `func TestAwaitRunLockOrPass(t *testing.T)`

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
- `internal/pairteardown#TestProbeQuietSeesTheGoDriverLock`
- `internal/pairteardown/doc.go`
- `pattern/PATTERN-pair-teardown.md`
- `docs/overview.md`

**Uses:**
- `internal/pauserecord#ListWaiting`
- `internal/pauserecord#WaitingMarker`
- `internal/runproc#List`
- `internal/runproc#Record`
- `internal/fabricengine#RunProcDir`
- `plan:internal/pairteardown#Teardown.registeredBattenAlive`
- `plan:internal/pairteardown#Teardown.driverRecordPID`
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
a live pause entry covers the slug (`pausedIn`, a hub or a slug entry of the hub's pause record, read as `HubSeamOptions`' `HubPaused` reads it; it also reports whether a hub entry is among them, which picks the resume form of the paused refusal below);
the local board entry reads `done` (`readClaim`), as D4 reads it;
the task worktree's status history holds a Finalize Done (`landed`) and batten is not running, which is no live registered batten (`battenAlive`, card 9), since batten still runs Land and deploy after Finalize Done.
These are the PATTERN entry's "paused, ended, owned by another machine, or done locally with batten no longer running".
A read error is returned, with nothing ended.

With the checks skipped, `EndSession` runs D4's other order: the run-lock wait, the refusal probe, `End`, the driver-lock re-check, the loop end, the loop end's own run-lock wait, the session end; there is no quiet wait.
The wait is a new substrate, `awaitOrPass func(ctx context.Context, slug string, paused, hubEntry bool, retry string) error`, taking both of `pausedIn`'s readings and the retry step, the step the operator runs again, which its refusal names; it is wired in `New` to `awaitRunLockOrPass` and faked in `fakeSubstrates.teardown`, so the sequence rows script its answer.
`EndSession` passes the retry step "the removal".
`awaitRunLockOrPass` waits, bounded by `runLockWait` through `awaitRunLockFree`, so a step doing local work under a pause, such as a merge under the parent lock, finishes before `End` ends the loop and the step with it.
At the bound it reads the holder, the newest unfinished step's pid (`shedverbs.RunningStepPID`) or, with no unfinished step record, the pid of the slug's live registered driver (`driverPID`, card 9), since a Go driver writes no step record and holds the run lock itself;
the holder is unknown only when neither is read, which leaves an unregistered process, such as one an older binary started.
It also reads the live waiting markers (`pauserecord.ListWaiting`, with `parentlock.ProcessAlive`) and whether a parent lock record names the slug, and `runLockHolderPasses` decides.
The holder passes, and is ended by `End` and the loop end, in two cases:
a live waiting marker of the slug names the holder's own pid (a holder waiting at the network gate, the incident's case);
or a live pause entry covers the slug and no parent lock record names it (a step waiting on its interrupted agent, which the loop's watch never kills under a pause).
The paused pass applies to a known and an unknown holder alike, since `End` and the loop end reach what they can and the driver-lock re-check refuses a holder neither reached.
A marker of another process never passes the holder, so a step merging under the parent lock is not ended on its own Finalize push's waiting at the gate.
Any other holder refuses with `runLockRefusal`'s error, wrapping `ErrDriverBusy`, with nothing ended:
a paused holder refused because a parent lock record names the slug, known or not, names its pid when known and the way forward of resuming the run so the step finishes its merge: `lyx tender resume` with no slug whenever a hub entry covers the slug, a slug entry beside it or not, since `lyx tender resume <slug>` is refused for a slug only a hub entry covers and beside a hub entry would leave the hub pause standing, and `lyx tender resume <slug>` only when slug entries alone cover it;
the no-slug form says that it lifts every live entry of the hub, other pairs' pauses and a pause held for another machine included, since the engine has no hub-only lift (`tenderengine.Resume` with an empty slug targets every live entry);
then the retry step again, since under the pause the step never ends and batten keeps the record;
the same refusal says that under a pause held for another machine, which batten keeps the record through only for a Finalize past its parent-side merge that finishes its landing, the way forward is running the retry step again once the holder ends, never lifting that pause;
any other known holder, unpaused with no parent lock record naming the slug, names its pid and first the way forward of `lyx tender pause <slug>` and then the retry step again, and second running the retry step again once that pid ends,
since the skip also reads true for an `ended` or `owned_elsewhere` verdict with no pause, nothing in the removal pauses that run and a Go driver holds the run lock for the run's whole life, so the pid does not end, while a pause passes the holder at the next attempt through the paused pass when no parent lock record names the slug;
an unpaused known holder with a parent lock record naming the slug names its pid and only running the retry step again once that pid ends, since a step merging under the parent lock finishes its merge, and a pause would turn the next attempt into the paused refusal whose resume brings this one back;
an unknown holder, unpaused, names the same `lyx tender pause <slug>` way forward first when no parent lock record names the slug, then the run lock's path and, as card 9's driver-lock refusal does by `goos`, the way forward of finding its holder in `lyx tender status`'s replaced-binary part or with `ps` on Linux, or ending the pair's `lyx loom drive` process in Task Manager on Windows, then the retry step again.

Without the skip the quiet wait and its refusal stand.
`busyError` keeps its blocker and drops "end it with `lyx reed down` in the pair", since card 12's `reed down` refuses exactly this running, unpaused run; its way forward becomes retrying once the driver has written its report, or `lyx tender pause <slug>` and then the removal again, which the skip then lets through, and it adds the slug's live owned processes (role and pid).
It reads them through a new substrate, `ownedProcesses func(slug string) ([]runproc.Record, error)`, wired in `New` to `runproc.List(fabricengine.RunProcDir(hub), slug)` and faked in `fakeSubstrates.teardown`, so the call-sequence row scripts the processes it names; a read error is named in the refusal in place of the list, which still wraps `ErrDriverBusy`.
`TestProbeQuietSeesTheGoDriverLock` (`internal/pairteardown/probequiet_integration_test.go`), which builds a bare `&Teardown{prime: ...}` and calls `busyError`, sets `ownedProcesses` to a function returning no records, so it keeps asserting the Go driver blocker.
`teardownEnded` passes `Verdict: ended`, so the reconcile's local removal of an ended entry takes the skip.

`TestRunLockHolderPasses` is a table over the pure rule: marker naming the holder's pid passes, with a parent lock record and without one; marker of another slug does not; paused with no parent lock passes; paused with a parent lock record refuses; unpaused with no marker refuses; a holder with a marker of another pid refuses, unpaused and paused with a parent lock record; unknown holder with no marker refuses; unknown holder, paused, with no parent lock record passes.
Each refusing row also asserts `runLockRefusal`'s wording, including the retry step it was given, which every refusal names: a paused holder with a parent lock record under slug entries alone names its pid, `lyx tender resume slug` and the retry under a pause held for another machine, and so does an unknown one without a pid;
the same refusal under a hub-only entry, and under a hub plus a slug entry, names `lyx tender resume` with no slug and that it lifts every live entry of the hub; an unpaused known holder with no parent lock record names its pid and `lyx tender pause slug`, and with a parent lock record names its pid and the retry step with no `lyx tender pause`; an unpaused unknown holder with no parent lock record names `lyx tender pause slug`, the run lock path and `lyx tender status` on `goos` linux, and Task Manager on `goos` windows.
`TestAwaitRunLockOrPass` (integration tag, in a file beside `TestProbeQuietSeesTheGoDriverLock`) runs over a real task worktree whose run lock the test holds with `mustHoldLock`, an unfinished step record carrying the test process's pid as the known holder, and `td.sleep` a counting no-op, so the `runLockWait` bound is spent without waiting.
Its rows call `awaitRunLockOrPass` itself, with the retry step "the removal":
a hub entry (`paused` and `hubEntry` true), a parent lock record naming the slug and no waiting marker of the holder returns an error wrapping `ErrDriverBusy` that names `lyx tender resume` with no slug and that it lifts every live entry of the hub;
the same with `hubEntry` false names `lyx tender resume slug`;
a live waiting marker of the slug carrying the holder's own pid, with a parent lock record present, returns nil;
`paused` true and no parent lock record returns nil;
a lock free at once returns nil with no sleep called.
`TestRunLockHolderPasses` pins the pure rule and the refusal's wording, and `TestAwaitRunLockOrPass` pins the holder, waiting-marker and parent-lock reads and the passing of both `pausedIn` readings into the refusal.
`TestTeardown_CallSequence` gains fake `paused`, `awaitOrPass` and `driverChecksSkipped` read answers and rows: an `ended` verdict, an `owned_elsewhere` verdict, a slug pause and a hub pause each call `awaitOrPass, refusal, endOwned, driverLock, kill, seize, await, end` with no `quiet`, the fake `awaitOrPass` seeing `hubEntry` false for the slug pause and true for the hub pause, and the retry step "the removal" on every row; a board entry done skips whatever batten reads; a Finalize Done with no live registered batten skips, and with a live one runs the quiet wait; an `awaitOrPass` that passes goes on to `endOwned`, and one that refuses with `ErrDriverBusy` returns it and calls no `endOwned`; a skipped-order refusal probe that refuses calls no `endOwned`; a skip-reading error is returned with no `endOwned`; a gone task worktree makes no skip reading.
Its existing busy-driver row, which asserts "lyx reed down" (`internal/pairteardown/teardown_test.go:189`), asserts `lyx tender pause slug` instead, and the role and pid of each process the fake `ownedProcesses` returns.
`TestTeardown_OriginSeams` gains a row: the `Teardown` seam's local removal carries `Verdict: ended`, so a busy driver under the fake substrates is skipped past and the removal runs.

Docs.
`doc.go` gains the skip readings, the pre-`End` run-lock wait and its two pass cases.
`pattern/PATTERN-pair-teardown.md`'s opening line becomes the declared text: "Every teardown of a pair goes through `internal/pairteardown`, which ends the pair's owned processes and then its reed session before any worktree is removed; its driver checks are skipped for a pair that is paused, ended, owned by another machine, or done locally with batten no longer running, never its run-lock wait while the task worktree is present, which then runs before the owned processes are ended."
Its quiet-wait bullet names the skip and the pre-`End` run-lock wait.
`docs/overview.md`'s `internal/pairteardown/` module row reads: quiet wait or the skip's run-lock wait, refusal probe, owned-process end, loop and reed session end, then fabric removal.

**ImpactSummary:** Request gains a field every existing caller leaves empty; EndSession's skipped path and busyError's way forward change inside pairteardown only, the one test asserting busyError's wording is updated here, and the one integration test calling busyError on a bare Teardown sets its new substrate.

**Commit:** 10: teardown skips the driver checks of a paused, ended, foreign or locally done run
