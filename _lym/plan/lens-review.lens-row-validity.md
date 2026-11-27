```yaml
lens: row-validity
outcome: reported
```

Row-validity review of plan cards 1–23: findings follow. Checking every row would have taken more calls than I had, so I checked the rows most likely to fail.

**MEDIUM — card 17, D8, `TestPushOwedForCatchUp` cannot be tier0 with only push and start faked.**
- `pushOwedForCatchUp` still calls `catchUpPair` (`internal/battencli/catchup.go:270`).
- `catchUpPair` runs `pairteardown.New(prime)` and `TaskWorktreeGone` (`catchup.go:198-211`). That is a real teardown whose `gone` read inspects the prime's repository.
- So the rows either spawn git, which tier0 forbids, or fail on a hand-built prime.
- The card's new signature lists no seam for "does the pair stand here", and none for the `HeldRecordsCommit` read the "held run" row needs.
- Fix: add a pair-locate seam to the re-sign, or make the test tier1 over a hub fixture.

**MEDIUM — card 19, D13, the `TestRemover` row does not reach the refusal.**
- `TestRemover` (`internal/fabriccli/remove_test.go:149`) drives `remover.run` over a fake teardown seam.
- A row where the fake returns `ErrLandingStands` only shows the error passing into the envelope. It would still pass if `Teardown.RemovePair` lacked the check.
- The criterion is a test that `lym fabric remove`, with and without `--force` and `--remote`, refuses before any mutation and leaves origin's board without `done` and origin's records without the held commit (tier1 or lowest that fits). No planned row checks origin's board or records.
- The `TestTeardown_CallSequence` row covers "records no call" but not origin state.
- Fix: add a tier1 row over a real pair and ledger, for example in `teardown_integration_test.go` or a fabriccli integration test.

**MEDIUM — card 19, `TestTeardownPushSeams_PushWakesThePairsPusher` cannot observe "starts the pusher".**
- The card wires `Push` straight to `runpusher.Start`. Under go test, `spawnPusher` either fails or forks the test binary: `selfexec.LymBinary` is the test binary (`internal/runpusher/start.go:160` says it "returns an error under go test").
- `teardownPushSeams` (`internal/battencli/wire.go:1786`) gets no start seam on this card.
- So the row either asserts an error path or spawns the test binary.
- Fix: give `teardownPushSeams` an injected start function, as `restartPusher(l, slug, start)` already takes, and list it.

**MEDIUM — card 19, `TestTeardown_CallSequence`'s landing rows need an unlisted read seam.**
- `Teardown`'s reads are injected function fields (`internal/pairteardown/teardown.go:115-139`, e.g. `paused`, `parentOf`), and the call-sequence test drives them by name.
- The card adds the landing-entry check to `Run` and `RemovePair` but lists no new `Teardown` field and no `pairteardown#New` edit.
- The rows would then hit a real `pushledger` read under whatever `HubPath` the hand-built prime carries. Its zero meaning is unstated.
- Fix: list a `landingStands` seam on `Teardown`, its production wiring in `New`, and the fake field.

**MEDIUM — card 11, D5, `TestParentPush`'s settle-only row calls an unexported function from an external package.**
- `TestParentPush` is declared in `internal/runpusher_test`.
- The row "`starter.start` spawning only a settle-only pusher" needs the unexported `starter`. Only `start_test.go`, in package `runpusher` (`internal/runpusher/start_test.go:3`), can reach it.
- Fix: make that row a row of `TestStart` and list `TestStart` and `startFake` on card 11.

**MEDIUM — card 7, issue #238, `TestSpawnStatusPush`'s red-first half cannot be red.**
- With `spawnDetachedPushFn` recording, no child runs on the merge base either.
- So "no failed-push record is written and the notifier is never called" holds before and after the change.
- Only the command-line assertion goes red.
- The criterion is that an owed failure "mails the orch on the merge base". The row as planned cannot observe that.
- Fix: run the real spawned child, or drive the merge base's `fabric push` child body over an unreachable origin. Otherwise state that the command line alone is the red assertion and drop the claim.

**MEDIUM — card 12, D6, a planned branch has no row.**
- The Intent makes a parent-owner read that fails on a transport error return `git-transport` (today: halt, `internal/landingshed/finalize_owner.go:79-82`).
- `TestFinalize_OwnerChecks` rows cover only the unreachable reconcile and the fetched-tips read. No row drives a failing `ParentOwner` seam.

**MEDIUM — card 4, used by cards 20 and 21: the ledger's clear time has no clock to inject.**
- `pushledger.Update` (`plan:internal/pushledger#Update`) takes no `now` but "records now" in `clear.json` when the last entry goes.
- `TestSyncStatus` pins `sync: 7m · 240s` "cleared seven minutes before". Card 4's test pins "the clear time set when the last entry goes".
- Neither can set the time without writing `clear.json` by hand, which bypasses the code under test, or comparing wall-clock time, which PATTERN-test-speed's "Time is injectable" forbids.
- Fix: give `Update` (or the package) an injected clock, and state it.

**LOW — card 5, D7, the authentication-refusal row cannot pin "one record and one notice".**
- In `TestRun` the fake `Push` returns a plain error (`internal/runpusher/run_test.go:77-85`).
- The record and the notice are written inside `fabricengine`'s classified push, not by the pusher. The row only shows the pusher stops.
- The criterion "stops at once with one record and one notice on an authentication refusal" needs a fabricengine-level row.

**LOW — card 5, `TestPusherDeps` and `pusherDeps` loading config.**
- `pusherDeps` now calls `fabricengine.LoadConfig` over the worktree's config dirs.
- `TestPusherDeps` (`internal/loomcli/pusher_test.go:26-35`) uses bare `t.TempDir()` locations with no anchor or config.
- The card states neither the error path nor that a missing config reads as defaults.

**LOW — card 6, existing `TestStart` rows.**
- `TestStart` uses the fixed hub `/hub` (`internal/runpusher/start_test.go:62`).
- Once `start` writes or removes ledger entries for `ifAhead`, the existing rows `IfAheadWithTheWorktreeAhead` and `IfAheadWithTheWorktreeLevel` (`start_test.go:102-103`) touch `/hub/...` unless the hub moves to `t.TempDir()`. The card doesn't say so.

**LOW — card 10, test economy and tier.**
- The `held_back` way-forward row is a pure-function check of `FailedPushWayForward`, placed inside tier1 `TestPushPrimeCodeTip`.
- No tier0 test of `FailedPushWayForward` exists (grep found none). The row belongs in a tier0 test.

**LOW — card 7, repeat.**
- `TestSpawnStatusPush`'s rows that pin the pair and prime pusher command lines repeat what `TestPushSpawnCommand` (`internal/fabricengine/spawn_test.go:49`) pins at tier0.

Tool calls:
1. Grepped the declarations of every test the cards name or create, and the packages of the spawn tests. The context lacked their locations, packages and existence.
2. Read the heads of `TestRunCommitsAhead`, `TestTeardown_CallSequence`, `TestRemover` and `TestPusherDeps`. The context lacked their fixtures and seam shapes.
3. Grepped for existing tests of `FailedPushWayForward`, `startRunPusher` and `loomStatusExtras`, plus `Teardown`'s seam fields, `TaskWorktreeGone` and selfexec's test behaviour. The context lacked them.
