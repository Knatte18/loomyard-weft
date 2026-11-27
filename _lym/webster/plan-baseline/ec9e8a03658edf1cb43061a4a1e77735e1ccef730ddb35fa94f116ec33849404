# Card 5 — pusher-interval

**Edit:**
- `internal/runpusher#Deps`
- `internal/runpusher#NewDeps` -> `func NewDeps(t Target, timing fabricengine.PushTiming) Deps`
- `internal/runpusher#Run`
- `internal/runpusher#pushUntilLevel`
- `internal/runpusher/doc.go`
- `internal/runpusher_test#TestRun`
- `internal/runpusher_test#runFake`
- `internal/runpusher#TestNewDeps_ConditionsStopOnAHandOverMarkOrARefusedPush`
- `internal/loomcli#pusherDeps`
- `internal/loomcli#TestPusherDeps`
- `internal/loomcli#loomCLI.pusherCmd`
- `internal/fabriclayout#PairPush`
- `internal/fabricengine#SyncOptions`
- `internal/fabricengine#PushPairAnchored`
- `internal/fabricengine_test#TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide`
- `internal/pairteardown#RunPush`
- `internal/pairteardown#PushRun`
- `cmd/lym#unboundedWaitFiles`

**Delete:**
- `internal/runpusher#noticeOf`

**Create:**
- `internal/runpusher#awaitInterval`
- `internal/runpusher#recordAttempt`
- `internal/runpusher#TestNewDeps_WaitWakesOnAPushNowMark`

**Uses:**
- `internal/fabricengine#PushTiming`
- `internal/fabricengine#Config.PushTiming`
- `internal/pushledger#Update`
- `internal/pushledger#Read`
- `internal/pushledger#RecordsKey`
- `internal/loomcli/doc.go`
- `internal/fabriclayout/doc.go`

**Intent:** D2 and D3: the pusher pushes a target at most once per interval in force, quietly, and keeps that timing in the ledger.

`Deps` loses `Notify` and `Sleep` and gains `Timing` (`fabricengine.PushTiming`), `HubPath`, `Key` (the target's `pushledger.Key`), `Now func() time.Time`
and `Wait func(ctx context.Context, d time.Duration, wake <-chan struct{}) error`.
`NewDeps` takes the timing, sets `Key` through `pushledger.RecordsKey` over the target's records worktree, `Now` to `time.Now`,
and `Wait` to a timer that also ends early on an `internal/fswatch` event on the ledger directory whose entry then carries `PushNow`, never within a second of the wait's start.
A zero `Timing`, as a hand-built `Deps` leaves it, reads as the defaults of card 1.

`Run`'s pass, through `pushUntilLevel`:
while the target is ahead and the conditions hold, it writes the target's entry when absent, with `FirstOwed` now and `IntervalS` the base interval;
every write of a records target's entry sets `Worktree` to the target's code worktree and `Slug` to the target's slug, empty for the prime;
`awaitInterval` waits out what is left of the interval in force since `LastAttempt`, unless the entry carries `PushNow`, which it clears and skips the wait once;
the attempt then reads the pause, runs the guard and pushes, as today, and `recordAttempt` writes `LastAttempt`.
An owed failure doubles `IntervalS` up to `Timing.Ceiling` and waits again; a push that lands returns it to `Timing.Interval`.
Level, the pass removes the entry unless it holds landings, `HeldRecords` or `Dropped`, which cards 11 and 19 settle;
an entry it keeps for `HeldRecords` or `Dropped` alone loses its `FirstOwed`, `LastAttempt` and `IntervalS`, so it reads held, not owed (card 4's `Entry.Owed`).
A pair ahead only by held commits reads level once card 8 makes `RunCommitsAhead` count no held commit, so its pusher exits after one pass keeping the entry.
A landing target whose pass pushed nothing and left every landing standing, with none held back and none owed, waits on its push-now mark between parent passes, not the interval;
a beam pair target waits the interval in force, since its own commits arrive through spawns that set no mark (card 7).
A refused push, a hand-over, a pause and a guard stop end the pusher as today and keep the entry.
A pusher started within the interval waits out its rest, since the attempt time is in the ledger, so the timing holds across pusher processes.
`PushUntilLanded` no longer serves the pusher; Worktree-Teardown's seam keeps it until card 19 deletes it.
No owed push posts a notice: `noticeOf` and the ceiling notice go, and D3's summary takes their place.

The pusher's pushes run with no transport backoff of their own (D2): `fabricengine.SyncOptions` gains `SingleAttempt`, which `pushClassified` reads so a transport failure returns owed after one attempt;
`fabriclayout.PairPush` gains the same field, which `PushPairAnchored` copies into its options, so every push through `SyncOptions`, a Bolt push included, can ask for it;
`pairteardown.RunPush` gains the same field and `PushRun` copies it, and `pusherDeps` sets it. Every other caller leaves it false and keeps today's backoff.

`pusherDeps` loads `fabricengine.LoadConfig` over the worktree's config dirs, logs `Config.PushTiming`'s floor lines once, and passes the timing to `NewDeps`; a config that fails to load logs one Warn and runs on the defaults. It no longer fills `Notify`.
`pusherCmd`'s `Long` text says the pusher retries an owed push once per interval in force, quietly.
`cmd/lym#unboundedWaitFiles` gains `internal/runpusher/run.go`, which holds the pusher's wait.
The doc's Running section says the same, with the interval, the doubling, the push-now mark and the ledger.

`TestRun` keeps its condition and stop rows and replaces its backoff rows, tier0, adding about a second to one tier0 run.
`runFake` drops its sleeps and notices and records the waits its `Wait` saw, over a real ledger under `t.TempDir()` and a fake `Now` the waits advance.
New rows: 60 s after a landed push; three owed failures waiting 60, 120 and 240 s then 300 s at the ceiling; back to 60 s after a push lands;
a second `Run` over the same ledger, started as a spawn starts it, with no push-now mark, 20 s after the last attempt, waiting 40 s; a `PushNow` entry pushing with no wait once and then waiting a full interval;
one push per interval after a transport failure, never a second within it; a zero `Timing` waiting the 60 s default;
a level pass removing the entry, and a level pass over an entry holding `HeldRecords` keeping it with its `FirstOwed` cleared, so it reads not `Owed`;
and an authentication refusal (D7), one push and the refusal returned with the entry kept and no wait after it.
The one failed-push record and one notice of that refusal are `fabricengine`'s, pinned by the subtest `RepeatedAuthRefusalsShareOneRecordAndOneMailAndALandedPushRemovesIt` in `internal/fabricengine/pushclasses_integration_test.go`.
`TestNewDeps_WaitWakesOnAPushNowMark`, tier1, pins that `NewDeps`' `Wait`, started with a long delay, returns once a `PushNow` mark is written to the target's entry, and not on a write that carries none.
`TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide`, tier1, gains a row: with `SingleAttempt` over an unreachable origin the push returns owed after one transport attempt, counted through `internal/gitexec`'s remote-attempt seam, which runs the row serially.
`TestNewDeps_ConditionsStopOnAHandOverMarkOrARefusedPush` and `TestPusherDeps` pass the timing to the new signature, and `TestPusherDeps` gains a row with no config that runs on the defaults.

**ImpactSummary:** Re-signs `NewDeps`, whose callers are `loomcli`'s pusher wiring and its tests, and adds one push field only the pusher sets; the pusher's retry and notice change, and nothing else reads them.
