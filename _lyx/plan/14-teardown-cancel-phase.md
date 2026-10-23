# Card 14 — teardown-cancel-phase

**Create:**
- `plan:internal/pairteardown#CancelLockWait` -> `const CancelLockWait = runLockWait`
- `plan:internal/pairteardown#CancelRequest` -> `type CancelRequest struct`
- `plan:internal/pairteardown#CancelResult` -> `type CancelResult struct`
- `plan:internal/pairteardown#Teardown.EndForCancel` -> `func (t *Teardown) EndForCancel(ctx context.Context, req CancelRequest) (CancelResult, error)`
- `plan:internal/pairteardown#Teardown.holdRecordsWriteLocks` -> `func (t *Teardown) holdRecordsWriteLocks(slug string) (release func() error, err error)`
- `plan:internal/pairteardown#Teardown.stopPairStrands` -> `func (t *Teardown) stopPairStrands(slug string) ([]string, error)`
- `plan:internal/pairteardown#liveStrandGUIDs` -> `func liveStrandGUIDs(rows []reedengine.DirectoryRow) (guids, names []string)`
- `plan:internal/pairteardown#cancelSurvivorsError` -> `func cancelSurvivorsError(slug string, survivors []runproc.Ended, goos string) error`
- `plan:internal/pairteardown#TestEndForCancel` -> `func TestEndForCancel(t *testing.T)`
- `plan:internal/pairteardown#TestHoldRecordsWriteLocks` -> `func TestHoldRecordsWriteLocks(t *testing.T)`
- `plan:internal/pairteardown#TestLiveStrandGUIDs` -> `func TestLiveStrandGUIDs(t *testing.T)`

**Edit:**
- `internal/pairteardown#Teardown`
- `internal/pairteardown#New`
- `internal/pairteardown#Teardown.driverLockBusyError` -> `func (t *Teardown) driverLockBusyError(slug, retry string) error`
- `internal/pairteardown#Teardown.endOwnedBeforeSession`
- `internal/pairteardown#fakeSubstrates`
- `internal/pairteardown#fakeSubstrates.teardown`
- `internal/pairteardown/doc.go`
- `pattern/PATTERN-pair-teardown.md`

**Uses:**
- `internal/fabricengine#HoldRecordsWriteLock`
- `internal/pairteardown#runLockWait`
- `internal/pairteardown#Teardown.holdPairLoopLock`
- `internal/pairteardown#Teardown.endReedSession`
- `internal/pairteardown#survivorNames`

**Intent:**
Add D4's cancel phase beside `EndSession`: it ends the run's owned processes and strands first and waits for the run lock after, since a cancel ends the holder instead of waiting for it.
It skips the quiet wait, the removal refusal probe and the pre-end run-lock wait a skipped teardown runs; what bounds that is that only `lyx batten cancel` calls it, from the prime, after its own refusals (D2).

`CancelLockWait` exports `runLockWait`, the run-lock bound, so batten's cancel waits for its own locks and its marker by the same bound without repeating the literal.
`CancelRequest` carries `Slug`, `Keep` (the pair stays) and `AfterEnd func() (busy string, err error)`, the caller's hook that takes its own locks once the processes are ended; `busy` names a lock path still held, and a hook that reports busy holds nothing, having released whatever it took before it returns, so the retry below takes no lock against its own process; a nil hook takes nothing.
`CancelResult` carries `Ended []runproc.Ended`, `StrandsStopped []string`, `SessionEnded bool` and `TaskWorktreeGone bool`.
The caller releases whatever its hook took on every return, a refusal included.
Every refusal the phase raises names `lyx batten cancel <slug>` again as the re-entry.

`EndForCancel` reads `t.gone` once, then runs in order, stopping at the first error:

1. `t.holdWriteLocks(slug)` takes, each through `fabricengine.HoldRecordsWriteLock` bounded by the `writeLockWait` field (`New` sets `runLockWait`), the prime's records write lock (`fabricengine.RecordsWorktree(t.prime)`) and, while the pair's records worktree (`fabricengine.RecordsWorktreePath(t.prime, slug)`) stands, the pair's; a spent bound releases what it took and refuses, wrapping `ErrDriverBusy`, naming the busy lock's path and the way forward: the prime's records write lock is shared by the hub, so its holder is often another run's or the orch's healthy commit, and the refusal says to see the holder with `lyx tender status` or `ps`, or in Task Manager on Windows, and run `lyx batten cancel <slug>` again once it lets go, ending the holder only when the same process keeps the lock across re-runs.
   Under them `t.endOwned(slug)` runs, and the locks are released as soon as it returns.
   A `*runproc.SurvivorsError` refuses with `cancelSurvivorsError(slug, survivors, t.goos)`: each survivor's pid and role, and the way forward D4 names, finding and ending them with `lyx tender status` and `ps` on Linux, or in Task Manager on Windows, then `lyx batten cancel <slug>` again; any other end error is returned wrapped.
2. With the task worktree present, `t.driverLockHeld(slug)`: held refuses with `t.driverLockBusyError(slug, "`lyx batten cancel <slug>`")`.
   Then `req.AfterEnd`; a non-empty busy runs step 1 once more, retaking both write locks, ending the owned processes and releasing the locks, and then this step again as a whole: the driver-lock probe with its refusal, then the hook; a lock still busy then refuses with `%w: <path> is still held after the run's processes were ended: a batten of <slug> or "lyx batten cleanup" may hold it; see the holder with "lyx tender status" or "ps", or in Task Manager on Windows, and run "lyx batten cancel <slug>" again once it is free`, wrapping `ErrDriverBusy`; any other hook error stops the phase.
3. With the task worktree present, `t.stopStrands(slug)`, wired to `stopPairStrands`: it lists the task worktree's reed directory, picks through `liveStrandGUIDs` every live strand, the `llm` child's driver strand and every arbiter and producer strand among them, stops each through shuttle's stop verb (`Runner.StopStrand`, PATTERN-shuttle-stop) and returns their names.
   With the task worktree gone there is no reed directory to read, so no strand is stopped by this step and step 6's end by name ends their panes (Shared Decisions).
4. With the task worktree present, `t.seizeLoop` kills the loop and its step tree and takes the loop lock.
5. With the task worktree present, `t.awaitRunLock` waits for the pair's run lock, bounded as `EndSession`'s wait is; a spent bound puts the pid file back.
   An error from step 4 or 5, whose text names retrying the removal, is returned wrapped as `%w; for this cancel, run "lyx batten cancel <slug>" again once the holder ends`.
6. `t.endSession(slug, gone)`: `Engine.Down` while present, the end by name once gone.
7. The loop lock is released: with `Keep`, through the release that withdraws the teardown's mark (`release(false)`), so the kept pair holds no teardown mark; without it, leaving the mark for the removal that follows.

`driverLockBusyError` takes the retried step, which `endOwnedBeforeSession` passes as `"the removal"`, so `EndSession`'s refusal reads as today; `survivorsBusyError` is unchanged.
`Teardown` gains the `holdWriteLocks` and `stopStrands` substrates and the `writeLockWait` duration, which `New` wires to `holdRecordsWriteLocks`, `stopPairStrands` and `runLockWait` (`stopPairStrands` builds the task worktree's reed engine through `t.reedEngine` and a shuttle runner over it, as batten's strand revive does); `fakeSubstrates` gains recorded fakes for both substrates, a `strands` answer, a `writeLockBusy` flag, a `driverLockHeldByProbe` list that answers each driver-lock probe in turn ahead of the `driverLockHeld` flag, and a `gone` answer that its `gone` substrate returns in place of today's constant false, and leaves every existing row's calls unchanged.

`TestEndForCancel` drives the phase over `fakeSubstrates` and pins the call order: present worktree `holdWriteLocks, endOwned, release, driverLock, afterEnd, stopStrands, kill, seize, await, end, release(ended=…)` with `Keep` releasing the mark and without it keeping it; a gone worktree (`gone` true) `holdWriteLocks, endOwned, release, afterEnd, end` with no driver-lock probe, strand stop, loop or wait; a hook busy once then free `holdWriteLocks, endOwned, release, driverLock, afterEnd, holdWriteLocks, endOwned, release, driverLock, afterEnd, stopStrands, …`; rows for a busy write lock (refusal naming the lock's path, `lyx tender status`, `ps`, Task Manager and `lyx batten cancel slug`), survivors (refusal naming each pid, `lyx tender status`, `ps` and `lyx batten cancel slug`), a held driver lock (naming `lyx batten cancel slug`), held at the first probe and, separately, free at the first probe and held at the second after a busy hook, a hook still busy (refusal naming the path, `lyx tender status`, `ps`, Task Manager and `lyx batten cancel`), a hook error, and a spent run-lock wait (naming `lyx batten cancel slug`), each asserting nothing after the refusal is called.
`TestHoldRecordsWriteLocks` is an untagged test over a `locationkit` prime with `writeLockWait` a few milliseconds, never zero, since `AcquireWriteLockWithin` with a zero wait reports busy before its first try: with no pair records worktree only the prime's lock is taken, with the pair's records directory present both are, and with the pair's lock held on another handle the call refuses naming that lock's path and holds neither lock afterwards.
`TestLiveStrandGUIDs` is a table: live rows are picked whatever their names, an `llm` driver, an arbiter and a producer strand among them, and dormant rows are not.

`doc.go` gains a Cancel phase section with the order above, and its Board claim and Local mode sections say that `Request.Snapshot` applies in origin mode and that `Request.LeaveClaim` leaves the claim to the caller (card 13); its Board claim sentence for a running record with no live batten now says the warning names `lyx batten cancel <slug>`, not `lyx board set-status <slug>`.
`pattern/PATTERN-pair-teardown.md`'s opening line becomes the text the overview's `## PATTERN changes` declares, and its bullets gain the cancel phase's order (owned processes and strands, the caller's locks, loop, run-lock wait, session end) and the claim the cancel's removal leaves to the cancel; its held-gone bullet names `lyx batten cancel <slug>` in place of `lyx board set-status`.
`PATTERN.md`'s one-line overview entry is unchanged, since its trigger, a teardown going only through `internal/pairteardown`, still holds.

**ImpactSummary:** `EndSession`'s driver-lock refusal builder gains a parameter its one caller fills with today's text; `New` wires two new substrates and a bound; no existing caller's behaviour changes.
