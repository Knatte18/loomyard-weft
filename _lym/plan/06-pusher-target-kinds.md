# Card 6 — pusher-target-kinds

**Edit:**
- `internal/runpusher#WorktreeFlag`
- `internal/runpusher#SlugFlag`
- `internal/runpusher#Target`
- `internal/runpusher#LockPath`
- `internal/runpusher#NewDeps`
- `internal/runpusher#starter.start`
- `internal/runpusher#newStarter`
- `internal/runpusher#pusherRequest`
- `internal/runpusher#TestStart`
- `internal/runpusher#startFake`
- `internal/runpusher/doc.go`
- `internal/loomcli#loomCLI.pusherCmd`
- `internal/loomcli#pusherDeps`
- `internal/loomcli#TestPusherDeps`
- `internal/beamcli#hubDobbyDepsStarting`

**Create:**
- `plan:internal/runpusher#TargetKind` -> `type TargetKind string`
- `plan:internal/runpusher#BoardTarget` -> `func BoardTarget(prime *lymcwd.Location) Target`
- `plan:internal/runpusher#LandingTarget` -> `func LandingTarget(prime *lymcwd.Location) Target`
- `plan:internal/runpusher#PruneLedger` -> `func PruneLedger(hubPath string, now time.Time) error`
- `plan:internal/runpusher#TargetRefused` -> `func TargetRefused(t Target) (bool, error)`
- `plan:internal/runpusher#TestPruneLedger` -> `func TestPruneLedger(t *testing.T)`
- `plan:internal/fabriclayout#PusherFlags` -> `type PusherFlags struct`
- `plan:internal/fabriclayout#PusherCommand` -> `func PusherCommand(flags PusherFlags) []string`
- `plan:internal/loomcli#TestStartRunPusher` -> `func TestStartRunPusher(t *testing.T)`
- `plan:internal/loomcli#TestPusherCmd_PushNowWakesAndExitsAtAHeldLock` -> `func TestPusherCmd_PushNowWakesAndExitsAtAHeldLock(t *testing.T)`
- `plan:internal/beamcli#restartHubPushers` -> `func restartHubPushers(prime *lymcwd.Location, start func(runpusher.Target) (bool, error))`
- `plan:internal/beamcli#TestRestartHubPushers` -> `func TestRestartHubPushers(t *testing.T)`

**Uses:**
- `internal/fabriclayout/doc.go`
- `internal/beamcli/doc.go`
- `internal/loomcli/doc.go`
- `internal/loomcli#startRunPusher`
- `internal/fabricengine#Bolt.HasUnpushed`
- `internal/fabricengine#RefusedPushStands`
- `internal/boardengine#Board.Sync`
- `plan:internal/pushledger#BoardKey`
- `plan:internal/pushledger#LandingKey`
- `plan:internal/pushledger#List`
- `internal/battencli#TestDobbyRuns_StartsThePusherOfOnlyAPairThisMachineOwns`
- `internal/battencli#TestWiredStartPusher_StartsThePrimesPusherAndAStandingPairs`

**Intent:** D1: the pusher gains the board's branch and the landing target beside the prime and a pair, and the dobby restarts all of them.

`Target` gains `Kind`, a `TargetKind` of `records` (the prime or a pair, the zero value), `board` or `landing`.
`BoardTarget` and `LandingTarget` build those over the prime; `LockPath` names `board.lock` and `landing.lock` beside the records locks, and `NewDeps` keys them `pushledger.BoardKey` and `pushledger.LandingKey`.
For the board, `Ahead` is the Bolt's `HasUnpushed` and the pause read `pauserecord.GateCovered`, so the board pusher yields to a pause as `lym board sync --deferrable` does.
The landing target's `Ahead` is "a landing entry stands", which card 11 pushes; this card gives it no push yet, so `pusherDeps` fills a landing `Push` that returns nil.
Every target's `Ahead` is also true while its entry holds a landing, so a beam pair whose code side went out by its own push stays on until card 11 settles its landings.

`TargetRefused` reads a target's refusal per D3, and `newStarter` and `NewDeps`' `Conditions` read it in place of a bare `RefusedPushStands`:
a records target through `RefusedPushStands` and the board through a refused-push record of the Bolt branch; the landing target, which has no push before card 11, reads unrefused here, and card 11 has it read the prime code branch's record.
A refusal of the prime's records branch therefore counts against the prime's records target alone, and a refused target starts no pusher on any tick (D5).

`fabriclayout.PusherCommand` is the one command line of the pusher verb, `loom pusher` with `--worktree`, `--slug`, `--board`, `--landing` and `--push-now` as the `PusherFlags` fields `Worktree`, `Slug`, `Board`, `Landing` and `PushNow` set them;
the flag names are `fabriclayout` constants, which `runpusher.WorktreeFlag` and `SlugFlag` now alias, so `fabricengine` and `boardengine` build the line without importing `runpusher`;
`pusherRequest` uses it, and so do the spawns card 7 turns into pusher starts.
`pusherCmd` reads `--board` and `--landing` to build the target, and with `--push-now` it sets the entry's `PushNow` through `pushledger.Update` before it takes the lock,
so a running pusher wakes on it and a pusher that finds the lock held exits having woken the one that holds it.
`pusherDeps` fills the board's `Push` with `Board.Sync` over the hub's board and no guard.

`starter.start` with `ifAhead` writes the target's ledger entry when the target is ahead and not refused, and removes it when the target is level and holds no landing and no `HeldRecords`, whoever pushed it (D3).
`PruneLedger` removes each entry whose `Worktree` no longer stands, so a tick drops the entry of a gone pair.
`restartHubPushers` restarts, through `start` and `RestartPusher`, the prime's records target, the board target and the landing target, then runs `PruneLedger`, logging a failure once per episode;
`hubDobbyDepsStarting`'s prime `Tick` calls it with its `startPusher` in place of today's lone prime restart.

Tests, tier0 unless named, adding about a second to one tier0 run (D11).
`TestStart` moves its hub under `t.TempDir()`, so its existing `IfAhead` rows touch a real ledger, and gains rows: a board target spawns `PusherCommand` with `--board` and no slug; a landing target with `--landing`; the pause and hand-over reads skip both;
`ifAhead` over an ahead target writes the ledger entry and over a level one with no landing removes it; a refused records target and a refused board target spawn nothing and write no entry.
`TestPusherDeps` pins the board target's pause seam and push seam and the landing target's level read.
`TestPruneLedger` pins that an entry whose worktree is gone goes and one whose worktree stands stays.
`TestRestartHubPushers` pins that `start` receives the prime's, the board's and the landing target's targets in that order, and that a failing start does not stop the others.
`TestStartRunPusher` pins that a loom resume's `startRunPusher` hands `start` the pair's target with its slug, and the prime's with none.
`TestPusherCmd_PushNowWakesAndExitsAtAHeldLock`, tier1, over a hub fixture with the target's pusher lock held by the test, pins that `lym loom pusher --push-now` writes the entry's `PushNow` mark and exits zero having pushed nothing.
The batten resume's restarts stay pinned by `TestDobbyRuns_StartsThePusherOfOnlyAPairThisMachineOwns` and `TestWiredStartPusher_StartsThePrimesPusherAndAStandingPairs`.

**ImpactSummary:** Adds two target kinds, three verb flags and one refusal read the starter and the conditions share; the prime and pair targets keep their paths, locks and spawns.
