```yaml
lens: guard-reach
outcome: reported
```

Task: guard-reach lens. For each guard, refusal, branch or test row the plan adds, I traced whether a real entry point can reach it and whether the guarded site recurs somewhere unguarded.

**Per guard, reachable or not**

- C1, invalid own marker is an error: reachable. A temp hub with an invalid `<name>-weft/.lyx-anchor`.
- C1, empty own marker falls through to Bolt: reachable.
- C2, `dobbyRuns` error and `dobbyWatchDirs` skip on an invalid marker: reachable through the row the card names.
- C2, `driverLockBusyError` falls back to the prime's lock path: **not reachable** (F2).
- C3, gone-weft fallback to D2: reachable, since the records worktree is deleted by hand.
- C4, `DobbyGeometry` returns a `NamedWorktree` error: reachable, through an invalid marker in a hand-built hub.
- C4, `Teardown.taskAnchor` falls back to the bare path: **not reachable** (F2).
- C5, a sibling whose Location read fails counts as still wired: reachable through an invalid sibling marker. No row is named, but it is a conservative branch, not a refusal.
- C6, Bolt-fallback stale guard and a `--subpath` mismatch from main-weft: reachable.
- C7, `refuseEmptyOwnMarker` on the fork point: reachable at the engine. Through the CLI it is reachable only where the cwd gate passes at the fallback anchor; the test drives the engine.
- C8, per-pair empty marker, migration and census keep/remove rows: reachable.
- C9, pull stop on a marker change, an empty marker or no stop: reachable inside `pullWeftArm`. **The guarded site recurs unguarded** (F1).
- C9, replaying-push stop and the `anchor_change` class: reachable. `retryWeftPush` runs only after a rejection, and the card's row builds one.
- C10, checkout refusals (local, remote-tracking, unfetched origin, warp-untracked target) and the drop-local refusal: reachable.
- C11, `anchor_stopped` from `originMover.move`; `judgeOrigin` lets a cancel through it; adopt's `stoppedBranches`: reachable. `originMover.move` is the only weft mover in the origin reconcile (`originreconcilemoves.go:410`).
- C12, promote refusal after `EndRun`/`MarkBeam`/`ClearRun`: reachable. `ResetRecordsToParent` has one production caller, `beamcli/deps.go:102`.
- C13, record refusals in add, reconcile, pull, replaying push, checkout and the origin mover: reachable with a hand-written record. The pull side also recurs unguarded (F1).
- C15, `BeginAnchorChange` refusals: reachable. That includes a held lock with a stale record pid (the test holds the lock and writes a dead pid), and a `B` absent from warp `HEAD` but tracked upstream (via `CloneSecond`).
- C15, the not-landed and landed recovery branches and the both-`.lyx` refusal: reachable by hand-built states.
- C16, `daemon.tick` gate: reachable. Passes, first passes and tidy start only from `tick` (`loop.go:273-282`; `SpawnPass` is called only from `passes.go:223` and `starts.go:122`), so nothing bypasses it.
- C16, `ensureOne` refusal: reachable.
- C17, every CLI refusal, the record exemption from the owner refusal, and the Windows refusal: reachable through fakes and the `GOOS` seam.
- C18, `Add` anchor validation and `EnsureAnchor`'s recover, equal, different and held-lock branches: reachable. `createSteps`' present-pair branch (`create.go:120`) comes before `MarkBeam`. The create's own dobby start may hit card 16's gate (F3).
- C19, adopt refusal before reconcile: reachable through the fake deps.

**Findings**

- **F1 — BLOCKING, design.** Cards 9 and 13 guard the weft pull only in `Fabric.pullWeftArm`.
  - A second weft pull, `Fabric.fastForwardWeft`, calls `f.PullWeft(opts)` with no marker comparison and no move-record check. Evidence: `internal/fabricengine/parenttips.go:270-290`.
  - It is reached from `Fabric.FastForwardPair` (`parenttips.go:258-260`), which landingshed's Finalize calls on the parent (`internal/landingshed/finalize.go:400`).
  - **Case 1:** a fast-forward onto an upstream that carries another machine's marker changes the parent's checked-out marker without rewiring it. D10 says every weft pull stops before that.
  - **Case 2:** it also runs while a move record stands, before the verb holds the parent lock. D7 step 1 says a weft pull of the worktree refuses then.
  - Fix: add `internal/fabricengine#Fabric.fastForwardWeft` to card 9's Edit list and run `weftAnchorStop` before `PullWeft`, skipping the weft pull with its warning on a stop. Add it to card 13's Edit list for `MoveInProgress`. Pin it with a row in the existing `FastForwardPair` test.
- **F2 — MEDIUM, design.** Card 2's `driverLockBusyError` fallback and card 4's `Teardown.taskAnchor` fallback cannot be reached. Card 4's invalid-marker teardown row therefore cannot pass, and card 2's claim that the fallback "is reached by card 4's teardown step" is false.
  - Upstream check: with an invalid own marker, `EndSession` → `waitQuiet` → `probeQuiet` calls `taskLocation` → `lyxcwd.ResolveWorktree` (`internal/pairteardown/teardown.go:662-663, 680-684`). Card 1 makes that call fail on an invalid own marker, so the error returns before `busyError` (`teardown.go:456-470, 490`).
  - On the lock path, `driverLockHeldIn` (`teardown.go:580-584`) errors the same way before `driverLockBusyError` (`teardown.go:301-312`).
  - Fix: drop the two fallbacks and the invalid-marker row, or build both texts from the Location `taskLocation` already resolved.
- **F3 — LOW, consistency.** Card 18 does not say when `Add` removes the create-at-anchor move record.
  - `Topology.add` "removes the record once the remaining create steps are done". But `createSteps`' `SpawnPush`, workspace regeneration and `startDobby` run after `AddPair` returns (`internal/beamcli/create.go:134-152`).
  - If the record still stands then, card 16's `ensureOne` gate refuses the create's own dobby start.
  - Fix: card 18 states that `Add` removes the record before it returns, ahead of `createSteps`' dobby start.

**Tool calls**

1. Grepped production call sites of `replayWeft`, `PullWeft`, `ResetRecordsToParent`, `Checkout`, the reset primitives and `reconcileRecordsBranch`. My context lacked the full set of weft movers needed to check whether the guarded site recurs.
2. Read `parenttips.go:225-300` and the `destroy.go` reset-keep function list. The second `PullWeft` site and the `DropLocal` and replay executors were not in the manifest.
3. Grepped `FastForwardPair` callers, the `PullWeft` body and the webster reset site. This established who reaches `fastForwardWeft`, and that webster's `ResetKeep` acts on a standalone worktree, not a weft.
4. Grepped `teardown.go` symbols and `createSteps`' present-pair flow. This traced card 4's teardown rows and card 18's re-run branch.
5. Read `teardown.go` `EndSession`, `endOwnedBeforeSession`, `waitQuiet`, `driverLockHeldIn` and `probeQuiet`. This confirmed the upstream `ResolveWorktree` error in front of both fallbacks.
6. Grepped the dobbyengine pass, tidy and spawn call sites. This confirmed that card 16's `tick` gate has no bypass.
