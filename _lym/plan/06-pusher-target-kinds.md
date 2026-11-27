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
- `internal/runpusher#TargetKind`
- `internal/runpusher#BoardTarget`
- `internal/runpusher#LandingTarget`
- `internal/runpusher#PruneLedger`
- `internal/runpusher#TargetRefused`
- `internal/runpusher#TestPruneLedger`
- `internal/fabriclayout#PusherFlags`
- `internal/fabriclayout#PusherCommand`
- `internal/loomcli#TestStartRunPusher`
- `internal/loomcli#TestPusherCmd_PushNowWakesAndExitsAtAHeldLock`
- `internal/beamcli#restartHubPushers`
- `internal/beamcli#TestRestartHubPushers`
- `internal/boardengine#Board.PusherSync`
- `internal/loomcli#TestPusherDeps_BoardPushCommitsTheBoardWrite`

**Uses:**
- `internal/fabriclayout/doc.go`
- `internal/loomcli/doc.go`
- `internal/loomcli#startRunPusher`
- `internal/fabricengine#Bolt.HasUnpushed`
- `internal/fabricengine#Bolt.HasUncommitted`
- `internal/boardengine#ApplySkipEnv`
- `internal/fabricengine#RefusedPushStands`
- `internal/boardengine#Board.Sync`
- `internal/pushledger#BoardKey`
- `internal/pushledger#LandingKey`
- `internal/pushledger#List`
- `internal/battencli#TestDobbyRuns_StartsThePusherOfOnlyAPairThisMachineOwns`
- `internal/battencli#TestWiredStartPusher_StartsThePrimesPusherAndAStandingPairs`

**Intent:** D1: the pusher gains the board's branch and the landing target beside the prime and a pair, and the dobby restarts all of them.

`Target` gains `Kind`, a `TargetKind` of `records` (the prime or a pair, the zero value), `board` or `landing`.
`BoardTarget` and `LandingTarget` build those over the prime; `LockPath` names `board.lock` and `landing.lock` beside the records locks, and `NewDeps` keys them `pushledger.BoardKey` and `pushledger.LandingKey`.
For the board, `Ahead` is true when the Bolt `HasUncommitted` or `HasUnpushed`, since a board write saves its files and leaves the commit to the sync it spawns;
with pushes off by `boardengine.ApplySkipEnv` it is `HasUncommitted` alone, and with git off it is false, so no board pusher runs endlessly.
`starter.start` with `ifAhead` reads the board through the same `Ahead`.
The board's pause read is `pauserecord.GateCovered`, so the board pusher yields to a pause as `lym board sync --deferrable` does.
The landing target's `Ahead` is "a landing entry stands", which card 11 pushes; this card gives it no push yet, so `pusherDeps` fills a landing `Push` that returns nil.
Every target's `Ahead` is also true while its entry holds a landing, so a beam pair whose code side went out by its own push stays on until card 11 settles its landings.

`TargetRefused` reads a target's refusal per D3, and `newStarter` and `NewDeps`' `Conditions` read it in place of a bare `RefusedPushStands`:
a records target through `RefusedPushStands` and the board through a refused-push record of the Bolt branch; the landing target, which has no push before card 11, reads unrefused here, and card 11 has it read the prime code branch's record.
A refusal of the prime's records branch therefore counts against the prime's records target alone, and a refused target starts no pusher on any tick (D5).

`fabriclayout.PusherCommand` is the one command line of the pusher verb, `loom pusher` with `--worktree`, `--slug`, `--board`, `--landing` and `--push-now` as the `PusherFlags` fields `Worktree`, `Slug`, `Board`, `Landing` and `PushNow` set them;
the flag names are `fabriclayout` constants, which `runpusher.WorktreeFlag` and `SlugFlag` now alias, so `fabricengine` and `boardengine` build the line without importing `runpusher`;
`pusherRequest` uses it, and so do the spawns card 7 turns into pusher starts.
`pusherCmd` reads `--board` and `--landing` to build the target: with either, it skips today's worktree resolution and target read, takes `--worktree`'s parent directory as the hub, since the board directory is no code worktree, and builds `BoardTarget` or `LandingTarget` over that hub's prime.
With `--push-now` it sets the entry's `PushNow` through `pushledger.Update` before it takes the lock,
so a running pusher wakes on it and a pusher that finds the lock held exits having woken the one that holds it.
Every write of a records target's entry, the push-now mark's and the starter's, sets `Worktree` to the target's code worktree and `Slug` to the target's slug, empty for the prime.
`pusherDeps` fills the board's `Push` with `Board.PusherSync` over the hub's board, its config passed through `ApplySkipEnv`, and no guard.
`PusherSync` commits the board's uncommitted files as `Sync` does, under the board's write lock, then pushes the Bolt through `Bolt.Push` with `SyncOptions.OwedUnrecorded` and `SyncOptions.SingleAttempt` (card 5) set,
so an owed failure leaves the commit owed, writes no failed-push record and mails nothing (D3, issue #238).
With pushes off it commits and pushes nothing; with git off it does nothing.

`starter.start` with `ifAhead` writes the target's ledger entry when the target is ahead and not refused, and removes it when the target is level and holds no landing, no `HeldRecords` and no `Dropped`, whoever pushed it (D3).
`PruneLedger` removes each entry whose `Worktree` no longer stands, so a tick drops the entry of a gone pair.
It keeps every entry holding landings, whose pending `done` writes D3 keeps until its parent push settles or drops them, and skips the board and landing keys, which name no worktree of their own.
`restartHubPushers` restarts, through `start` and `RestartPusher`, the prime's records target, the board target and the landing target, then runs `PruneLedger`, logging a failure once per episode;
`hubDobbyDepsStarting`'s prime `Tick` calls it with its `startPusher` in place of today's lone prime restart.

Tests, tier0 unless named, adding about a second to one tier0 run (D11).
`TestStart` moves its hub under `t.TempDir()`, so its existing `IfAhead` rows touch a real ledger, and gains rows: a board target spawns `PusherCommand` with `--board`, the prime's `--worktree` and no slug; a landing target with `--landing`; the pause and hand-over reads skip both;
`ifAhead` over an ahead pair target writes the ledger entry with the pair's code worktree as `Worktree` and its slug as `Slug`, and over a level one with no landing removes it; a refused records target and a refused board target spawn nothing and write no entry.
`TestPusherDeps` pins the board target's pause seam and push seam and the landing target's level read.
`TestPusherDeps_BoardPushCommitsTheBoardWrite`, tier1, over a hub fixture with a bare origin, adding about three seconds to one tier1 run, drives the board target's `Ahead` and `Push` through `pusherDeps`:
a board write left uncommitted reads ahead, and the push commits it and puts it on origin;
over an unreachable origin the push commits it, returns owed and writes no failed-push record, and the target still reads ahead;
with `BOARD_SKIP_PUSH` set through `t.Setenv`, a committed but unpushed Bolt reads level; the variable is process-global state, so the test names it and runs serially.
`TestPruneLedger` pins that an entry whose worktree is gone goes, one whose worktree stands stays, a gone-worktree entry holding a landing stays, and the board and landing entries stay.
`TestRestartHubPushers` pins that `start` receives the prime's, the board's and the landing target's targets in that order, and that a failing start does not stop the others.
`TestStartRunPusher` pins that a loom resume's `startRunPusher` hands `start` the pair's target with its slug, and the prime's with none.
`TestPusherCmd_PushNowWakesAndExitsAtAHeldLock`, tier1, over a hub fixture with the target's pusher lock held by the test, pins that `lym loom pusher --push-now` writes the entry's `PushNow` mark and exits zero having pushed nothing,
and that the exact arguments `boardengine.SpawnSync` builds, `--board` with `--worktree` the board directory, run with the board pusher's lock held, exit zero having pushed nothing and written no mark.
The batten resume's restarts stay pinned by `TestDobbyRuns_StartsThePusherOfOnlyAPairThisMachineOwns` and `TestWiredStartPusher_StartsThePrimesPusherAndAStandingPairs`.

**ImpactSummary:** Adds two target kinds, three verb flags, one refusal read the starter and the conditions share, and the board's commit-and-push `Board.PusherSync`; the board reads ahead over uncommitted files too, and the prime and pair targets keep their paths, locks and spawns.
