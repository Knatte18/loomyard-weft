# Fixer report — plan, round 3

All changes are in `_lyx/plan/09-creel-deliver-marked-probe.md`, card 9's Intent.

## Changes

- **F1 (BLOCKING), fixed differently.**
  Premise confirmed: `Paused` is read at the start of `locate` (`deliver.go:242`), the probe's `Send`, journal writes and `noteStall` follow, and the worker calls `clock.After` only after the probe returns (`deliver.go:146`), so both waits return with the worker not parked, and `tick` (`deliver_test.go:38`) can lose its advance.
  Added a postcondition bullet stating where `awaitMarkedProbe` and `awaitTyped` return and the 2 s to 8 s backed-off wait, and the rule that a subtest that went through either wait never follows it with a bare `tick` or `probeWaits(before + n)`; a later flip or jump takes `seen` before, `mark` after, calls `awaitMarkedProbe` again and then reads `typed` or calls `awaitTyped`.
  `HoldMutesOtherSendersLetsTheOwnerPassAndEndsAtExpiry` gets the named shape: `e.clock.Advance(31 * time.Minute)` between `seen` and `mark`, `awaitMarkedProbe`, then `awaitTyped` for the full `want`, with the `tick(time.Second)` after the jump dropped, since one probe types one item (`deliver.go` `probe`) and the hold ends by the clock alone.
  Not made as proposed: the shapes for `BusyReceiverIsRemindedAtItsNextIdleEdge` and `PauseTypesNothingUntilLifted`.
  Both post once after `e.start` to a receiver with nothing pending, so `Scheduler.wake` (`deliver.go:89-95`) creates the worker with an empty wake channel, its first probe snapshots after the post, and `probeWaits(1)` returns with the worker parked; the card now exempts such a post, so these two keep their `probeWaits(1)`, flip and `tick(time.Second)`, which meet a parked worker.
  Reshaping them as proposed would have been harmless for them, but the same exemption is needed anyway for `PostToALiveIdleStrandIsRemindedOnce` and `UnansweredReminderBacksOffToTheCeilingAndANewMessageResets`, where the proposed shape cannot keep the exact back-off timing the reminder loop asserts (see the self-check).
  The parked-worker qualifier reads: a worker whose last recorded wait was awaited by `probeWaits(1)` after `e.start`, by `driveProbeWaits` or by a `tick` that a parked worker entered, with no wake and no advance past that wait since.
- **F2 (MEDIUM), fixed as proposed, with one subtest's shape changed and one member added.**
  Added that an inject after `e.start` wakes the worker as a post does (`verbs.go:397`), and that `holdAfterAReminderProbeReadTheDraft` being unchanged covers only its pre-start post.
  `StallCleared...` is unchanged: it waits on `waiter.C`.
  `PersistingStall...` keeps `probeWaits(before + 1)`, with the reason that the worker is parked at the inject: `Start` gives it an empty wake channel, `Hold` sends no wake (no `wakeLocked` in `Hold`, `verbs.go:537`), and the last tick returned on its recorded wait.
  None of the three calls `awaitMarkedProbe`; the `note.At.Equal(e.clock.Now())` reason is attached to `PersistingStall...` alone.
  `ExpiryNames...` takes `before` before its `InjectWait` and `probeWaits(before + 1)` after it, because keeping its bare ticks can leave a probe in flight at its final `Advance(time.Minute)` and time out `awaitOutcome` (raised by the self-check).
  `InjectPostedMidWaitIsChargedFromItsPostingNotFromThePreviousProbe` is the other inject-after-start subtest that counts probe waits: it gets the same `before` / `probeWaits(before + 1)` around its `InjectWait`, since its bare ticks after the inject can chain a lost advance into its final `tick(8 * time.Second)`.
- **F3 (LOW), fixed as proposed.**
  The bound sentence now says three passes cover a single post, each further post before the call can add one pass, and `HoldMutes...` is bounded by four passes, 32 s.
  The freshness sentence now says two passes advance 16 s, which is not shorter than `stallFreshness` (`pool.go:83` drops a stall only when the age is strictly greater), so the assertion fails at two passes.
- The negative-check rule now says that, with several posts, `seen` is taken before the first and `mark` after the last (self-check).
- "This covers the subtests with more than one strand reader" now reads "The reshaping by check covers…", since the inserted text had displaced its referent.

## Self-check

- fix-consistency, MEDIUM, the parked-worker qualifier contradicted the `PersistingStall...` and `InjectPostedMidWait...` reasons: fixed as proposed, the qualifier was widened to a `tick` entered by a parked worker and to `driveProbeWaits` followed by an advance short of the wait.
- fix-consistency, LOW, `seen` and `mark` placement with several posts unstated: fixed as proposed.
- fix-consistency, LOW, D6 in `_lyx/discussion/decision-record.md` still says the negative check waits for the marked probe and the wait after it: deferred, see below.
- fix-consistency, LOW, D6 covers posts only while the card now covers injects: deferred, see below.
- fix-claims, MEDIUM, `ExpiryNames...` keeping its ticks can leave the final minute advance firing nothing: fixed as proposed (`before` / `probeWaits(before + 1)` around the inject).
- fix-claims, LOW, "reaches `stallFreshness`" false for one pass and irrelevant to two of the three subtests: fixed with own wording, the reason is narrowed to `PersistingStall...`'s clock-equality check.
- fix-proposal, MEDIUM, same as the fix-claims `ExpiryNames...` finding: merged, fixed as above.
- fix-affected-set, MEDIUM, `PostToALiveIdleStrandIsRemindedOnce` and `UnansweredReminderBacksOffToTheCeilingAndANewMessageResets` follow a reshaped wait with bare ticks and had no shape: fixed with own change, by the fork's second option: a single post after `e.start` to a receiver with nothing pending since creates the worker with an empty wake channel and is unchanged, as a pre-start post is; this also covers `ReplyToAStrandWithNoWorkerIsRemindedOnce`, the first posts of `LogByID...`, `BusyReceiver...` and `PauseTypes...`, and the card names the later posts that the rule still reshapes.

## Disputed

None.

## Deferred

- D6's wording in `_lyx/discussion/decision-record.md` ("waits for the marked probe and the wait after it", and its scope to posts only) disagrees with card 9's postcondition and its inject coverage.
  The decision record lies outside this round's write surface (`_lyx/plan`), so it needs the loop owner or a discussion round to update; the card's text is the sound mechanism.
