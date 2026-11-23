---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed-differently
    reason: "Instead of reading a branch whose run origin's board ended as held, the push deletes the tracking ref of a branch gone from origin with its entry ended, so TipsOnOrigin's existing absent-ref rule reads it held. A local read cannot tell a branch origin no longer has from one it still has, and the ended set marks a run ended when it is done (boardengine/replay.go:286), so the proposed read would also pass a branch still on origin that carries unpushed commits."
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed-differently
    reason: "Same change as F5: the push's checkOrigin deletes the stale tracking ref of a branch gone from origin with its entry ended, so RunCommitsAhead's existing absent-ref rule reads it level from the next tick on. A read keyed on the ended set alone would also read a branch still on origin as level, which the push itself still pushes (pushclassified.go:672-678)."
  - id: F14
    disposition: fixed
  - id: F15
    disposition: fixed
---

# Plan round 1: fixer report

## Changes

- **F1** (card 3): `retryWeftPush` runs `primeTakeovers` on any records worktree that is the prime's with a non-zero `here`, whatever `runSlug` is.
  `takeoverStop` stays for a pair's records worktree alone.
  The intent, bound and ImpactSummary are reworded to match.
  `TestRejectedRecordsPush_ReadsTheTakeover` gains a prime row that pushes with a slug naming the run not taken over and expects a `*TakeoversError` naming the taken-over run, with nothing pushed.
  Card 4 says the `*TakeoversError` branch serves every prime push, including one that names a run, and its ImpactSummary says so too.
- **F2** (card 10): `envkit#nilLegal` and `loomcli#intentionallyNil` are added to **Edit:**, with an intent line covering both maps.
- **F3** (card 8): the pair's `StartIfAhead` runs inside the `!inputs.OwnedElsewhere && view.PairPresent` condition.
  A new bound says a pair owned elsewhere gets no pusher from the tick, and that `HandOverOwnedElsewhere` pushes such a run's owed commits.
- **F4** (card 6): the resolve failure's message ends in the way-forward clause, and the `TestResolvePersistentPreRun_SkipsSlugGroups` row asserts that clause.
- **F5** and **F13**, fixed differently (cards 1, 2 and 10):
  - In card 1, `pushJob.checkOrigin` (now under **Edit:**) deletes the local tracking ref of a branch gone from origin whose entry ended, in every mode.
  - Card 1's test gains rows that assert the ref is deleted, on the records side and on the code side.
  - In card 2, both reads name that deletion as what turns the stale-ref case into the absent-ref case they already read level and held.
  - Card 10's Call bullet says `Level` then reads such a pair held, so the removal goes on.
- **F6** (card 3): the `TestHandOverTips` rows with `PrimeRecords` set build the request with `Pair` nil.
- **F7** (card 7): `tendercli#resumeCmd` and `tenderengine#Deps` are added to **Edit:**, and both packages' `doc.go` to **Uses:**.
  A new tender intent line covers both.
- **F8** (card 1): the `TestPushPairAnchored_OwedUnrecorded` Create target is dropped.
  The new cases are now `OwedUnrecorded` variants of the existing owed rows plus new rows in `TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide`.
  Card 11's "Held by" line now names that test.
- **F9** (card 8): the `TestRestartPusher` Create target and its intent line are dropped.
- **F10** (cards 5 and 8): `RestartPusher` is moved to card 5 as `plan:internal/runpusher#RestartPusher`, with its intent paragraph and `internal/logger#EpisodeSet` under **Uses:**.
  Card 8 lists it under **Uses:** and no longer creates it.
- **F11** (card 1): `fabricengine#StatusPushLockWait` is added to **Edit:**, with an intent line rewording its doc.
- **F12** (card 7): `StartPush` becomes `func() bool` in `statuscommit.Deps` and in both `commitStatusDeps`, filled with `runpusher.Start`'s started value.
  A failed start is logged and read as false.
  `pusher_started` carries that value, and `TestCommitRecordsVerb_Success` asserts both true and false.
- **F14** (card 8): `internal/beamcli/doc.go` moves from **Uses:** to **Edit:**, with an intent line naming the prime's pusher restart at the start of its `Tick`.
- **F15** (card 3): the `TestIsPushOwed` paragraph rewords the `pushowed_test.go` file header.

`lym loom validate-plan` reports `ok: true` after every change.

## Self-check

- fix-consistency, MEDIUM: `RunCommitsAhead` covered only the records side while `TipsOnOrigin` covered both. Fixed differently: the ended-set sentences are gone, and both reads now rest on card 1's tracking-ref deletion, which covers both sides.
- fix-consistency, LOW: the "so" clause in card 5's `Target` paragraph was stale after F1. Fixed: the clause is reworded.
- fix-claims, MEDIUM: the ended-set read in `TipsOnOrigin` and `RunCommitsAhead` was too wide. Merged with fix-proposal 2 and fixed differently, as F5/F13 above.
- fix-claims, MEDIUM: "never started for it" did not hold before a fetch. Fixed: the sentence now says later ticks read it level once one pusher has met the branch.
- fix-proposal, MEDIUM (F8): `TeardownLockHeldPastTheWaitOwesTheRecordsPush` is in `TestPushLock_BoundedWaitGivesUpAndUnboundedWaitBlocks`. Fixed: that test is added to card 1's **Edit:**, and its row's variant is planned there.
- fix-proposal, MEDIUM (F5/F13): the same widening the fix-claims lens found. Merged and fixed differently.
- fix-proposal, LOW (F1): `PushRun` no longer said whether a push naming a taken-over run still returns `ErrHandedOver`. Fixed: card 4 says it does, and a `TestPushRun_HandsOverOnATakeover` row asserts it.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
