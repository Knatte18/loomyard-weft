---
verdict: APPROVED
---

No finding survived consolidation.
The range is the merge-base diff against `main` (no rework round exists), 13 commits with no `Pattern-Departure:` trailer.
The tier-0 gate over `orchcli`, `orchengine`, `reedengine`, `creelcli` and `loomcli` passed (2666 passed, 0 failed, 2 skipped).

Focus questions settled:

- Card 3: every strand-liveness reader in reed (`directory.go`, `lifecycle.go` twice, `strand.go` twice) takes `engineAliveIDSet`; `namerepair.go`, `paneoptions.go` and `selvagepane.go` keep `aliveIDSet`.
- Card 8: the only writers and removers of `CycleRequestPath` are `RequestCycle`, `WithdrawCycleRequest` and `takeMarkerUnlocked`; the first two take the state lock through `lockMarker`, and `takeMarkerUnlocked` runs under `RemoveCycleRequestIf` or inside `saveCycleStart`'s `UpdateJSON`.
- Card 13: `stop` takes `start.lock` with a bounded wait; `restartWatcher` takes it with a try, re-reads state, strands and watch lock under it, and spawns only for a tracked, live strand with no watcher.
- Card 10 stencils: the reworded stencils drop the reply request and keep the soft `defer`, matching the card's Intent.
- Test economy: each added behaviour has a pinning test, including `withdraw`, `held_by_pause`, `strand_elsewhere`, the held `start.lock` hint and the re-check under the lock.

## Rejected

- A test that `RequestCycle` and `WithdrawCycleRequest` hold `StateLockPath` (code-tests lens, MEDIUM): the plan's card 8 does not ask for it, and a deterministic test of a lock held across a call needs a timing seam or wall-clock wait the repository's test rules forbid; the compare-and-remove behaviour is pinned by `TestCycleRequest_MarkerChangedDuringStart` and the `RemoveCycleRequestIf` rows.
- No test for the held `start.lock` hint, the re-check under the lock, or an empty `strand_elsewhere` for an untracked strand (code-tests lens, LOW): `TestStatus_RestartsADeadWatcher` carries the rows `AHeldStartLockSpawnsNothingAndHints` and `AStrandRemovedBeforeTheLockSpawnsNothing`, and `TestStatus_Envelope` asserts `strand_elsewhere` in its expected maps.
