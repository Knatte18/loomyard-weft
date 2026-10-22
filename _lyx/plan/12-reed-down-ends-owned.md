# Card 12 — lyx reed down in a pair ends the owned processes first, and refuses a running unpaused run

**Create:**
- `internal/pairteardown#ErrRunRunning`
- `internal/pairteardown#Teardown.EndBeforeSessionDown`
- `internal/pairteardown#TestEndBeforeSessionDown`
- `internal/reedcli#reedCLI.endPairProcesses`
- `internal/reedcli#TestEndPairProcesses`

**Edit:**
- `internal/reedcli#reedCLI`
- `internal/reedcli#reedCLI.downCmd`
- `internal/reedcli#TestDownCmd_WaitsForTheWatchdogOnlyWhenItEndedTheServer`
- `internal/reedcli#fakeStrandOps`
- `internal/reedcli#fakeStrandOps.Down`
- `internal/pairteardown/doc.go`
- `internal/reedcli/doc.go`

**Uses:**
- `internal/pairteardown#Teardown.pausedIn`
- `internal/pairteardown#Teardown.awaitRunLockOrPass`
- `internal/pairteardown#Teardown.registeredBattenAlive`
- `internal/pairteardown#Teardown.endOwnedProcesses`
- `internal/runproc#SurvivorsError`
- `internal/pairteardown#fakeSubstrates.teardown`
- `internal/pairteardown#New`

**Intent:**
D4's `reed down` half.
Today `reed down` in a pair kills the session of a running run, whose strands batten then revives; this withdraws that.
`reed down` runs no origin reconcile and makes no network call, so it never waits at the gate: it judges no branch, takes no owner verdict and starts, resumes or lands nothing (`PATTERN-origin-reconcile` constraint).

`EndBeforeSessionDown` reads only local state, in order, stopping at the first that holds:
a live pause entry covers the slug (`pausedIn`; a run owned elsewhere reaches this as the entry its owner pause wrote, card 13);
the local board entry reads done (`readClaim`);
the status history holds a Finalize Done (`landed`) and no registered batten is alive;
the run is not running: the quiet probe reads no driver lock held, the loop lock free and no live driver strand, and no registered batten is alive.
There is no `stopped` reading, since that is a reconcile verdict.
On any of these it runs the `awaitOrPass` substrate (card 10) with the retry step "`lyx reed down`" and both of `pausedIn`'s readings, whether a pause entry covers the slug and whether a hub entry does, so a refusal under a hub pause names `lyx tender resume` with no slug, then `endOwned` for the slug, and returns the ended processes; a refusal of the run-lock wait is returned as it is, its retry step already `lyx reed down`, and a `*runproc.SurvivorsError` is returned wrapped with each survivor's pid and the way forward of retrying `lyx reed down` once it is gone.
Otherwise it returns an error wrapping `ErrRunRunning`, naming the slug and the way forward `lyx tender pause <slug>`, then `lyx reed down` again, with nothing ended; a run whose live loom driver strand alone makes it running is refused the same way.

`reedCLI` gains the seam `endProcesses func(ctx context.Context) ([]runproc.Ended, error)`, nil selecting `endPairProcesses`, which resolves the invoking worktree from `c.anchorPath`, returns nothing for the prime (no pair, so nothing is ended and `down` runs as today), returns a resolution error as the envelope's error with `Down` not called, and otherwise builds `pairteardown.New` over the hub's prime location and calls `EndBeforeSessionDown` for the worktree's slug.
`downCmd` calls it before `Down`; an error is the envelope's error with `Down` never called and the session live; otherwise `Down` runs as today and the envelope carries `ended_processes`, the list of `{pid, role}`.
Reed's session end itself is unchanged (D9).

`TestEndBeforeSessionDown` drives the method through `fakeSubstrates.teardown`, the fake `awaitOrPass` seeing the retry step "`lyx reed down`" on every row that calls it, with rows: a covering slug pause and a covering hub pause, the fake `awaitOrPass` seeing `hubEntry` false for the first and true for the second, an owner pause entry, a local done entry, a Finalize Done with no live batten, and a not-running run each call `awaitOrPass` then `endOwned` and return what it ended; a Finalize Done with a live registered batten, and a running unpaused run, a live driver strand included, refuse with `ErrRunRunning` naming `lyx tender pause slug` and call no `endOwned`; a refusing `awaitOrPass` returns its error naming `lyx reed down` and calls no `endOwned`; survivors return the wrapped error naming the pid.
`fakeStrandOps` gains a `downCalls` count that its `Down` increments.
`TestDownCmd_WaitsForTheWatchdogOnlyWhenItEndedTheServer` sets the seam on every row (a no-op answering no processes) and gains rows: a refusing seam yields the error envelope with `downCalls` zero; a seam answering two processes yields `ended_processes` beside `session` with `downCalls` one.
`TestEndPairProcesses` (integration tag, over a `hubforge` hub with one pair) installs a pause gate (`pauserecord.InstallGate`) over the hub's record holding a live hub entry, whose `Sleep` fails the test, so any network call would be held and fail it: `endPairProcesses` from the pair's anchor reads the pause, ends nothing registered and returns an empty list without sleeping, which pins that `reed down` makes no network call; from the prime's anchor it returns nothing without reading any pair.
The two docs state the readings, the refusal and the envelope key, and `downCmd`'s `Long` help text, which today documents an unconditional kill, gains the pair readings, the refusal of a running unpaused run naming `lyx tender pause <slug>`, and the `ended_processes` key.

**ImpactSummary:** reed down gains a pre-step in a pair only; the prime's down and reed's session end are unchanged.

**Commit:** 12: lyx reed down ends a pair's owned processes first and refuses a running unpaused run
