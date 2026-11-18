# Pause holds no push lock and skips pairs with no reed session

## A pause holds a pusher only before it takes a push lock

A push the pause gate held used to wait at its first git call while it already held `fabric.push.lock`.
Every other pair's push then failed with `push lock busy ... held by pid <n>` until the held process was killed.

Every taker of a push lock now calls `pauserecord.Admit` before it takes its first lock and stays admitted until it releases the last.
That covers the weft's `fabric.push.lock` and the board checkout's push lock.
This is the scheme `gitrepo.HoldRemoteRefs` already uses for the remote-refs lock.
A covered process waits at the gate with no lock held, and a pause that starts while a pusher holds the lock reads its operation in flight.
The sites are:

- `coalesceFlagged`, admitted on each acquire of its loop: every pair push, detached or synchronous;
- `coalescePush`: `Bolt.Sync`;
- `lockWeftForReplay`, before the weft write lock;
- `takeoverBranch.land`;
- `Bolt.pullFirst`;
- `retryBoltPush`, before the board write lock.

The new gate operation is named `push-lock`.

A detached push child's gate defers instead of waiting, so it now defers at the gate before the lock, when no side has been pushed yet.
Once admitted, no later call of the process meets the gate, so the deferral never fires mid-step.
The `pushed` map that tracked which sides a step had reached was dead code after this change and is removed.
`pushDeferral` now puts back the flag of every owed side.

## Pause, status and revive skip a pair with no reed session

After a hard power-off, a pair's run records can still read running while its reed session is gone.
`lym tender pause` and `lym tender status` then failed on that pair's session listing with reed's `no reed session` refusal.
A pair with no reed session has no running turn to interrupt.
Pause, status and revive now match `reedengine.ErrNoSession` from `Pair.Sessions` and handle that pair as follows:

- the pair goes under `skipped` with reed's refusal as the reason, and the other pairs are still read;
- pause keeps the entry it already recorded;
- `StatusReport` gains a `skipped` part for this, and the prime is handled the same way.

## Design decisions

- **Dobby needs no change to start no driver for a paused pair.**
  `DobbyRuns` reads the pause record into the act rule's `PauseCovers`, and `runfsm.Act` waits a covered run.
  `planPass` gives no pass to a run whose verdict does not act, on any trigger.
  `TestPlanPass`'s `VerdictPauseCovers*GivesNone` rows pin that, so no new test was added for it.
  In the incident, the pause of that pair was failing on the missing reed session, which the second change fixes.
- **The teardown lock is still taken before the gate.**
  `PushPairAnchored` takes the pair's teardown lock shared before the push-lock loop admits it.
  A held push therefore still blocks that pair's own teardown, but never another pair's push.

## Tests

- `TestCoalescePushBothAt_HeldByAPauseWithoutThePushLock`:
  while a pause holds a push, the push lock is free for another pusher; after the lift the push lands both sides.
- `TestCoalescePushRequest_CoveredCallDefers`, renamed row `CoveredCallDefersBeforeThePushLock`:
  a covered detached child exits at the gate with the lock free and both flags back.
- `TestPushDeferral_PutsBackEveryOwedFlag` replaces the table that covered partly pushed steps.
- `TestSync_DeferrableExitsUnderACoveringPause`, renamed row `PlainCoveredWaitsThenReachesTheBoard`:
  a plain board sync under a pause now waits once at the gate before the board's push lock, then reaches the board.
- `TestPause`, new row: a pair with no reed session is skipped, the pause keeps its entry, and the other pair is interrupted.
- `TestStatus`, new subtest: a pair and a prime with no reed session are listed under `skipped`, and the other pair's session is still read.
