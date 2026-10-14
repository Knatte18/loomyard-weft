# Card 4 — hibernate-engine-watch

**Create:**
- `internal/hibernateengine/watch.go`
- `internal/hibernateengine/watch_test.go`
- `internal/hibernateengine#Watch`
- `internal/hibernateengine#waitForTrigger`
- `internal/hibernateengine#triggerReading`
- `internal/hibernateengine#landFinished`
- `internal/hibernateengine#confirmed`
- `internal/hibernateengine#awaitWake`
- `internal/hibernateengine#advance`
- `internal/hibernateengine#finish`
- `internal/hibernateengine#pauseGrace`
- `internal/hibernateengine#confirmInterval`
- `internal/hibernateengine#confirmRereads`
- `internal/hibernateengine#wakeWindow`
- `internal/hibernateengine#wakePoll`
- `internal/hibernateengine#triggerPollBase`
- `internal/hibernateengine#triggerPollCeiling`
- `internal/hibernateengine#TestWatch`

**Edit:**
- `internal/hibernateengine/doc.go`
- `cmd/lyx#unboundedWaitFiles`
- `pattern/PATTERN-daemon-wakeups.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/hibernateengine#fakeWorld`
- `internal/hibernateengine#newFakeWorld`
- `internal/hibernateengine#fakeWorld.deps`
- `internal/hibernateengine#Deps`
- `internal/hibernateengine#Request`
- `internal/hibernateengine#updateRequest`
- `internal/hibernateengine#liftOwned`
- `internal/battenrecipe#NameWorktreeTeardown`
- `internal/pauserecord#SleepWatch`

**Intent:**
Add the daemon's sequence, `Watch(deps)`, which the hidden `lyx hibernate watch` verb (card 5) runs: it waits for the trigger, pauses the hub, confirms the pause, hibernates the machine, detects the wake and lifts its own pause entry.
The daemon sends no key and no text of its own: every interrupt and resume line is tender's, under its existing guards (`PATTERN-guarded-interrupt`, `PATTERN-pane-text-injection`).
The fixed timings are the constants above, with no config key.

**Ownership.**
`Watch` refuses an agent caller and a non-Linux platform as `Submit` does.
It then reads the request under its lock and acts only for a non-terminal request whose `DaemonPID` and `DaemonStart` equal `SelfPID` and `SelfStart`; with no request, a terminal one or another daemon's, the mutate returns `errUnchanged`, so it returns at once with a nil error, having paused nothing, hibernated nothing and written no request file.
`advance` moves the request to a phase with a reason under the lock, and reports false, writing nothing, when the request is no longer this daemon's or has gone terminal, as after `lyx hibernate cancel`; every step below calls it and stops quietly on false, leaving the lifting to `Cancel`.
`finish` writes a terminal phase and its reason the same way.
Each phase change logs one Info line, and each poll at most one Debug line (`internal/logger`'s level policy).

**Waiting.**
`waitForTrigger` reads the trigger through `triggerReading` and returns once it fires.
- `now` fires at once.
- `after <slug>` reads `RunStatus(slug)`: a status not found ends the request `failed` with `the batten status of <slug> is gone`; `landFinished` (state `done`, or current row `battenrecipe.NameWorktreeTeardown`, the one row after Land, in any state) fires, so a run whose teardown halts after Land still fires; a run reading `running` before Land waits with the reason `waiting for <slug> to finish Land, now at <producer>`; a run halted before Land, in any other state, keeps waiting with the reason `the batten run of <slug> halted at <producer> (<state>); run "lyx hibernate cancel" to withdraw the request`.
- `after-all` lists `RunSlugs` and reads each status: any run reading `running` waits with the reason naming the slugs in flight; otherwise it fires, at once when there are no runs, and a halted or awaiting run counts as out of flight; a listed slug whose status is missing is skipped.
A reason that changes is written through `advance(PhaseWaiting, reason)`, so `lyx hibernate status` shows it.
The poll sleeps from `triggerPollBase`, doubling to `triggerPollCeiling` while the reason is unchanged and returning to the base when it changes (`PATTERN-daemon-wakeups`).
Bound: this wait has no deadline, since the operator cancels a request whose run halted.

**Firing.**
`advance(PhasePausing)`, then `Pause(tenderengine.PauseRequest{Grace: pauseGrace, Added: ...})`, which `lyx pause` runs too, covering every pair and leaving the orch to its live watcher.
The `Added` callback records the entry as `Owned` (`EntryRef{Scope, Caller, PausedAt}` from the entry) under the request lock before the grace, so `Cancel` mid-pause finds it; it writes `Owned` whatever the phase, and is called only when the pause adds an entry.
A report with `AlreadyPaused` records no owned entry, so neither the daemon nor `Cancel` lifts that pause.
A report with `ResumedDuringGrace` ends the request `cancelled` with the reason `the pause was lifted before the hibernate`.
The `Added` callback keeps the `EntryRef` it builds in a variable of `Watch` before it calls `updateRequest`, so a failed write of `Owned` still leaves the daemon holding its entry, as card 1's `Added` contract provides.
A `Pause` error ends the request `failed` with the error, lifts the owned entry and queues one notice; when the request records no `Owned` entry but the callback kept one, that kept `EntryRef` is lifted through `Lift` instead, so the notice reads that its pause was lifted.
The lift and the notice run even when the `failed` write itself fails, and `Watch` then returns that write error.

**Confirming.**
`advance(PhaseConfirming)`, then readings: one right after the pause, then up to `confirmRereads` re-reads, each after `Sleep(confirmInterval)`.
Each reading takes `TenderStatus()` and `LandLockHeld()`; `confirmed` holds when all of these hold at once:
- a live entry of scope `pauserecord.ScopeHub` stands in `status.Live`, owned or not;
- every session in `status.Sessions`, pair and prime alike, the orch included, reads `idle-done`, `idle-stalled`, `idle-fresh`, `asking` or `dead`; a session is listed while its pane is live, so one whose process is gone can read `dead`, and does not withhold; `busy` of any cause, background work included, `unknown` and any other state withhold it;
- `status.NetOps` is empty;
- the land lock is not held.
`status.Holds`, the parent locks, is not read, since a paused run keeps its parent lock by design.
The first reading also withholds while the pause report's `StillBusy` or `Refused` is non-empty.
A reading with no live hub entry ends the request `cancelled` at once with no hibernate, the operator's `lyx resume` having lifted the pause.
A reading error is logged and counts as unconfirmed.
With no confirmed reading after the last re-read, ten minutes in all, the request ends `failed` with the last withholding reason, the owned entry is lifted and one notice is queued; `Hibernate` is never called.

**Hibernating.**
Right before the call, `advance(PhaseHibernating)` re-checks that the request is still this daemon's and not cancelled, and a fresh `TenderStatus()` must show a live hub entry, else the request ends `cancelled` with no hibernate.
A `pauserecord.SleepWatch` is observed with `Now()` and `Awake()` right before `Hibernate()`.
A non-zero exit (a `Hibernate` error) ends the request `failed` with it, lifts the owned entry and queues one notice.
`systemctl hibernate` returns once the job is queued, so `awaitWake` then observes the watch every `wakePoll`, and a sleep the watch records ending after the call is the wake; with no such sleep within `wakeWindow` of awake time after the call, the request ends `failed` with `no wake was seen within five minutes`, lifts the owned entry and queues one notice.

**Waking.**
`advance(PhaseResuming)`, then `liftOwned`, which lifts by identity, so an owned entry already resumed, with another hub entry added meanwhile, lifts nothing and leaves that entry live; a lift that lifts nothing is not an error.
The request ends `done`, and one notice is queued through `Notify`: `[lyx hibernate] The machine woke from hibernation; ` followed by `its pause was lifted.`, `its pause had already been lifted.` or `it owned no pause, so the hub pause that stood before the request still stands.`.
A failure notice reads `[lyx hibernate] The hibernate request failed: <reason>; its pause was lifted.`, or `...; it owned no pause to lift.`.
Both are one line and never start with `/`, the notice queue's own rules.

**Tests.**
`TestWatch` drives `Watch` over `fakeWorld` (card 3), one row per case, each seeding a request whose daemon identity is the world's own:
- `now`, run by the orch caller: the hub is paused once with grace ten seconds, the owned entry recorded in the file before the grace, `Hibernate` called once after a confirmed reading, the wake seen, the owned entry lifted from the record, phase `done`, and exactly one notice saying the pause was lifted;
- the owned entry resumed and another hub entry added before the wake, from `onSleep` after the call: nothing is lifted, the other entry stays live, phase `done`, and the notice says the pause had already been lifted;
- a hub entry live before the request: the pause reports `AlreadyPaused`, no owned entry is recorded, the wake lifts nothing and that entry stays live;
- the same with a pair session reading `busy` at the first reading, so a re-read follows, and that entry resumed from `onSleep` before it: phase `cancelled` and no `Hibernate` call;
- the operator resuming the owned entry the same way, a first reading withheld by a busy session and the entry resumed from `onSleep`: phase `cancelled`, no `Hibernate` call;
- the pre-call check, two sub-rows through `onStatus` on the call after the confirmed reading: the owned entry resumed, and the request cancelled through `Cancel`; each ends with no `Hibernate` call, the first `cancelled` by the daemon and the second left `cancelled` by `Cancel` with `Watch` writing nothing more;
- a pause report with `ResumedDuringGrace`: phase `cancelled`, no reading and no `Hibernate` call;
- a `Pause` error: phase `failed` with the error, the owned entry recorded through `Added` lifted, one failure notice; and a sub-row whose `onPause` hook replaces the request file with a directory of the same name, so the `Added` write fails: `Watch` returns an error, the hub entry is gone from the pause record's live list, and the one failure notice says its pause was lifted;
- sessions listed reading `idle-stalled`, `idle-fresh`, `asking` and `dead`, an external caller running `Watch`: confirmed at the first reading;
- a `TenderStatus` error at the first reading, clearing at the second: no `Hibernate` before the second reading, one after it;
- withholding, one sub-row each for a pair session reading `busy`, a session reading `busy` with cause background work, a session reading `unknown`, the orch (a prime session) reading `busy`, a first-reading `StillBusy` strand, a first-reading `Refused` session, a net op in flight and the land lock held: no `Hibernate` call while the condition holds, and one after it clears, at the first reading where it is gone;
- a parent lock held in `status.Holds` alone confirms at the first reading;
- 120 unconfirmed re-reads: phase `failed`, the clock advanced by exactly ten minutes, the owned entry lifted, one failure notice and no `Hibernate` call;
- a `Hibernate` error: phase `failed`, the owned entry lifted, one failure notice;
- no sleep within five minutes of awake time after the call: phase `failed`, the owned entry lifted, one failure notice;
- `--after <slug>` reading `running` at `Run-Shed`, then `running` at `Land`, then `blocked` at `Worktree-Teardown`: no pause until the third reading, the waiting reason written while it waits, then the full sequence; with `running` at `Run-Shed` repeated over five readings before the change, the trigger sleeps are 1s, 2s, 4s and 8s, then back to 1s after the reason changes;
- `--after <slug>` already `done`, and already `running` at `Worktree-Teardown`, two sub-rows: each fires on the first reading;
- `--after <slug>` reading `blocked` at `Run-Shed`: phase stays `waiting` with the halt reason, read from the file by an `onSleep` hook that then cancels the request through `Cancel`, after which `Watch` returns with no pause;
- `--after <slug>` whose status goes missing while waiting: phase `failed` with the reason, no pause;
- `--after-all` with one run reading `running` and then `done`, plus one `blocked` run: fires once the first reads `done`; with no runs it fires at once; and a sub-row listing, beside one run reading `done`, a slug whose status the missing-slug switch reports as not found: it fires on the first reading and the request does not end `failed`;
- with no request, with a terminal one and with a request whose daemon pid is another's: no pause, no `Hibernate`, and the file absent or byte-for-byte unchanged;
- an agent caller and a non-Linux GOOS are refused, with no pause.

`cmd/lyx#unboundedWaitFiles` gains `internal/hibernateengine/watch.go`, and `PATTERN-daemon-wakeups.md`'s "What counts" list gains the hibernate daemon's trigger wait, which polls from one second and doubles to thirty seconds while its reading is unchanged.
`internal/hibernateengine/doc.go` gains the daemon's section: ownership, the triggers, the firing, the confirmation, the hibernate and wake windows and the notices, with the bounds above.
`cmd/lyx/testdata/test-budget.yaml` raises `internal/hibernateengine` to 4.

**ImpactSummary:** new code in the card-3 package; the wakeups guard's file list gains one entry and its background one line.

**Commit:** 4: hibernate: the daemon's trigger wait, pause, confirmation, hibernate and wake
