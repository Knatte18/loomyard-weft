---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/09-creel-deliver-marked-probe.md:38
    summary: awaitMarkedProbe and awaitTyped return while the receiver's worker is mid-probe with no wait registered, and the card leaves the tick steps that follow them in the same subtests unspecified, so the stale-count race of #744 returns one step later
    fix: In card 9's Intent, state the postcondition of both waits, and add a rule for what follows them. Postcondition - awaitMarkedProbe returns when the receiver's worker has begun the probe after the marked one (the pause read at mark+2), and awaitTyped returns when the typed line is visible, which is before that probe journals its outcome and registers its next clock.After; in both cases the worker is not parked, and after the helper's passes its backed-off wait is 2 s to 8 s, not 1 s. Rule - a subtest that went through either wait never follows it with a bare tick(d) or probeWaits(before + n); each later step that flips a fake or jumps the clock (BusyReceiverIsRemindedAtItsNextIdleEdge after SetIdle(true), PauseTypesNothingUntilLifted after SetPaused(false), HoldMutesOtherSendersLetsTheOwnerPassAndEndsAtExpiry at tick(31 * time.Minute) and the tick(time.Second) after it) takes seen before and mark after the flip or jump, calls awaitMarkedProbe again, and then reads typed or calls awaitTyped for the lines it asserts. Name that shape in the card for those three subtests. Where a step really needs a parked worker, only a worker started by e.start whose first wait was awaited with probeWaits(1), or one driven by driveProbeWaits, qualifies. The fix adds no target, so validate-plan --overlay is not needed.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/09-creel-deliver-marked-probe.md:53
    summary: the card does not say how the three stall subtests that call holdAfterAReminderProbeReadTheDraft and then inject after e.start are reshaped, and a negative-check reading breaks their clock-equality assertion and the 16 s stall freshness the card says no reshaped check runs under
    fix: In card 9's Intent, add one sentence that names StallClearedUnderAHoldIsNotSeededIntoALaterWaiter, PersistingStallIsToldOnceToALaterWaiterAndEachChangeOnce and ExpiryNamesNoStallLastReadBeforeTheBound, and says that holdAfterAReminderProbeReadTheDraft being unchanged covers only its own pre-start posts, not the InjectWait each of these makes after e.start. State the rule for them. They never call awaitMarkedProbe, because its advance of 8 s to 24 s moves e.clock.Now() past the probe that told the waiter (PersistingStall asserts note.At.Equal(e.clock.Now())) and lies at or past stallFreshness. Their wait after the inject is either kept as probeWaits(before + 1), with the reason that the worker is parked when the inject is made so no stale wake or in-flight probe exists, or replaced by a receive from waiter.Progress or waiter.C, with the clock unmoved. Also make the card's inject coverage explicit, since the Intent says post or interrupt only while an inject wakes the worker the same way.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/09-creel-deliver-marked-probe.md:44
    summary: the three-pass bound is stated for any interleaving but holds only for one wake, and the sentence on stall freshness calls two passes an excess when two passes advance exactly 16 s
    fix: In card 9's Intent, rewrite the Bound sentence to say the three passes cover a single post, and that each further post made before the helper is called can add one wake-driven probe and so one more pass, so HoldMutesOtherSendersLetsTheOwnerPassAndEndsAtExpiry, which posts twice before its first call, is bounded by four passes, 32 s, still far under its 30-minute hold. Replace the sentence "Two or three passes already exceed the 16 s stall freshness" with one that says two passes advance 16 s, which is not shorter than stallFreshness (the staleness test in pool.go is strictly greater than), so the shorter-than assertion already fails at two passes.
---

### [BLOCKING] The wait helpers leave the worker mid-probe, and the follow-up ticks race it

**Location:** `_lyx/plan/09-creel-deliver-marked-probe.md:38` (the loop's end condition), with the continuations at `internal/creelengine/deliver_test.go` lines 285-298, 470-486 and 487-513 as they stand now.

**Issue:** The helper ends the moment `PausedCalls(receiver)` reaches `mark+2`.
`Paused` counts at the start of `locate`, so the worker has just begun the probe after the marked one, which still has its `Idle` read, `Send`, `journalReminded` and `noteStall` ahead, and only then calls `clock.After`.
The decision record asks the test to wait for the marked probe "and then for the wait that follows it", which is a parked worker; the card's end state is one probe later and not parked.
`awaitTyped` has the same tail: the fake records the line at `Send`, and the worker journals before it registers its next wait.
In the old shape `probeWaits(1)` returned with the worker parked, so the `tick` after it was safe.
Now the continuation steps do not stay safe.
`tick` takes `before`, calls `Advance`, then waits for `before+1` waits.
When the worker registers its wait after `before` was read, `Advance` fires nothing for it, the pending `probeWaits(before + 1)` is satisfied by the worker's own late wait, and the test goes on with the flip unseen.
Three subtests follow a card-9 wait with such a step:

- `BusyReceiverIsRemindedAtItsNextIdleEdge` flips `SetIdle(true)` and ticks 1 s; a probe that read `Idle` before the flip leaves no reminder, and the assertion `want one reminder` fails.
- `PauseTypesNothingUntilLifted` flips `SetPaused(false)` and ticks 1 s, with the same failure.
- `HoldMutesOtherSendersLetsTheOwnerPassAndEndsAtExpiry` ends its positive check with `awaitTyped` and then runs `tick(31 * time.Minute)` and `tick(time.Second)`. If the 31-minute advance is lost, the worker's wait is due 1 s past the new clock, the 1 s tick fires it once, the probe types only the webster notice, and the expected reminder of two messages is missing.

Independent of the race, the 1 s tick no longer fires anything: the helper's passes advance the worker's back-off, and its wait is 2 s to 8 s by then.
The card's only guidance is that "a subtest whose later assertion counts probe waits or ticks takes the passes the helper made into that count"; that adjusts a count and says nothing about the duration or the in-flight probe.
The window is realistic: `awaitTyped` polls every millisecond, and the worker's journal writes sit between the line being visible and its next `clock.After`; this is the case `-count=1000 -cpu=1,2,4` exercises, so the card's own Verify would be flaky as specified.

### [MEDIUM] Stall subtests built on the pre-start helper and an inject after start are not covered

**Location:** `_lyx/plan/09-creel-deliver-marked-probe.md:53`

**Issue:** The card says a subtest or helper that posts before `e.start`, such as `holdAfterAReminderProbeReadTheDraft`, is unchanged and never calls `awaitMarkedProbe`.
The two helpers that post before `e.start` in the file are `holdAfterAReminderProbeReadTheDraft` and `orchStrandIsRemindedAndTypedInto`, and the claim holds for both: `Start` sets the hook and calls `wake` for each pending receiver, which creates the worker with a fresh, empty wake channel, so the first probe snapshots every earlier post, and the card changes neither helper's pause or pane reads.
But three subtests call `holdAfterAReminderProbeReadTheDraft` and then call `InjectWait` after `e.start`: `StallClearedUnderAHoldIsNotSeededIntoALaterWaiter`, `PersistingStallIsToldOnceToALaterWaiterAndEachChangeOnce` and `ExpiryNamesNoStallLastReadBeforeTheBound`.
`PersistingStall...` then waits with `probeWaits(before + 1)`, which is the shape D6 and the Intent's "every subtest ... after a post made once `e.start` has run" cover, and its `told` closure asserts `note.At.Equal(e.clock.Now())`.
Under the card's negative-check rule the helper would advance the clock 8 s to 24 s after the probe that told the waiter, which breaks that equality, and its advance reaches `stallFreshness` (16 s in `pool.go`), against the card's own statement that no reshaped negative check runs under such a window.
The card does not say which rule governs these subtests, and it names posts and interrupts but not injects, which wake the worker the same way (`wakeLocked` in the inject path).

### [LOW] The pass bound and the freshness sentence do not match the code or the card's own cases

**Location:** `_lyx/plan/09-creel-deliver-marked-probe.md:44`

**Issue:** The bound of three passes counts an in-flight probe, one wake-driven probe whose pause read precedes `mark`, and the marked probe's wait.
That is right for one post.
The worker's wake channel holds one signal, and every post made while a probe is running refills it, so two posts before the first call (as `HoldMutesOtherSendersLetsTheOwnerPassAndEndsAtExpiry` makes) can put a second wake-driven probe before `mark` and add a fourth pass.
The card states the bound for a subtest whose only live worker is the receiver's, which that subtest satisfies, so the stated bound of three is too low for it.
Separately, `freshStallLocked` drops a stall only when the age is strictly greater than `stallFreshness` (16 s), so "two or three passes already exceed" is wrong for two passes (exactly 16 s); the shorter-than assertion still fails at two passes, so the conclusion stands and only the wording needs correcting.

## Rejected

- Liveness of the loop for one receiver: not a defect. A timer the worker has parked on was registered before the previous advance and fired by it (every probe wait is at most 8 s, the advance is 8 s, and `After` records the wait and the timer under one lock), or it was registered after it and raised the count to the next pass's threshold, so no interleaving of the post's wake-driven probe with the `seen` and `mark` reads leaves the worker parked with no pass to fire it. The loop can time out only when the worker retires, which the negative checks it is used for do not do (unread mail stays pending).
- The marker claim that only the probe and `interruptTarget` read a strand's pause: confirmed by the two `Paused` call sites in `deliver.go` and `verbs.go`.
- `seen` before the post and `mark` after it: sound; a wait counted into `seen` from a probe started by the post's own wake would have made pass 1 wait for a wait that never comes, and the card avoids that.
