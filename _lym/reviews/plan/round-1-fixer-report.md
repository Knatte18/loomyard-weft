---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: ruling
    reason: "Parent ruling 'A protected-branch refusal keeps the rejection path' carried out in card 7."
  - id: F3
    disposition: fixed
  - id: F4
    disposition: ruling
    reason: "Parent ruling 'Push verbs refuse an unset run record seam before the push lock' (option a) carried out in cards 4, 7 and 8."
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed-differently
    reason: "Planting no status record would break TestReturnToPreviousTransition, which decodes the planted status and asserts its machine (internal/fabricengine/codetipsettle_integration_test.go:128-133). Card 12 moves plantRunTransition into a shared helper file instead, so codetip_integration_test.go drops both shed imports and every caller keeps the planted record."
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed-differently
    reason: "The proposed reason names 'a test's own hub path under the sequential rule', but after F8 no test sets RunLocks to a path of its own, since TestReconcileOrigin_PrimeLockHeld is removed. The restated reason drops that clause and keeps the bound: only the installer and SetRunLocks set the seam."
  - id: F11
    disposition: ruling
    reason: "Parent ruling 'Fabric asks a status codec verdict instead of decoding shed records' (option a) carried out in cards 2, 3, 9, 10 and 11."
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
  - id: F14
    disposition: fixed
---

# Plan round 1 — fixer report

The plan validator (`lym loom validate-plan`) passes after every change below.

## Changes

- **F1** (cards 7, 9): the two snapshot tests are now in `fabricengine_test`.
  Each reads its snapshot through a new `export_test.go` accessor: `PushClassifiedSnapshotForTest` for card 7 and `ReconcileJudgeSnapshotForTest` for card 9.
  Each Tests bullet says why package `fabricengine` cannot hold the test.
- **F3** (card 7):
  - Added `OutcomeRetriesExhausted`. `outcomeClassOf` maps `gitrepo.ErrPushRetriesExhausted` to it, ahead of the transport case.
  - The transport rule never pushes an exhausted push again, and `JudgePush` records it as `transport`.
  - Added a `TestPushSide` row and the `TestOutcomeClassOf` rows.
  - The ImpactSummary drops its class count.
- **F5** (card 3):
  - Replaced the search with the review's cross-package grep and recorded its result on this base: no file needs `hubforge.InstallRequiredSeams`.
  - The hits outside package `fabricengine` build their hubs through `hubforge`. `weftskippush_integration_test.go` mentions `CloneAndWire` only in a comment.
  - The package-`fabricengine` hits reach no seam read.
  - The line on which cards read the seams now says "cards 4, 9, 10 and 11".
- **F6** (card 12): added Create lines for the review's fixtures.
  - The moved rows' bodies also call `pairFixture`/`newPairFixture`, `pushPair`/`newPushPair`, `requireTrackingRefPrunedWithUpstreamSet`, `originHasBranch`, `countingBackoff`, `holdWeftWriteLock`, `landOn`, `statusJSON`, `tagsOf`, `withBoardStatus` and `failedPushesOf`. I read each body and added these too.
  - The fixtures bullet names them all, with `failureFixture`'s methods.
- **F7**, fixed differently (card 12): `plantRunTransition` moves to a new shared helper file, `transitionplant_helper_test.go`, and keeps planting the status record.
  - `codetip_integration_test.go` drops both shed imports and the header sentence about holding the helper.
  - The owner-guard rows read the trailer and the reflog, never the machine (`pushclassified.go:417-447`, `478`), so none of them moves.
  - Not made: planting with no status record, which would break `TestReturnToPreviousTransition` (`codetipsettle_integration_test.go:128-133`).
- **F8** (cards 4, 11):
  - Card 4's `TestInstallRequiredSeams` drops the run-lock check and no longer unsets `RunLocks`.
  - Card 11 removes `TestReconcileOrigin_PrimeLockHeld` and edits `TestInstallRequiredSeams` to add the run-lock check.
  - Card 11 also deletes the prime-lock row of `TestReconcileOrigin_Moves`.
- **F9** (card 4): the test plants its own transition in `fabriccli_test`.
  - It writes the status with two history rows and commits it with the transition and warp-SHA trailers, then pushes and fetches.
  - After the self-check, the planted status sits at the `TreeRunFileRels(slug)` tree path that `OriginCodeTip` reads.
- **F10**, fixed differently (card 11): `uncontainedWriteAllowlist` moves to Edit, and the entry's reason is restated with its bound. The reason is under F10 in the frontmatter.
- **F12** (card 7): `internal/fabricengine/doc.go` moves from Extends to Edit, and the outcome-class sentence is rewritten.
- **F13** (card 4): the engine asks `RemoteOnly` with `hasOrigin` alone first and stops at a false answer. After the self-check, nil tips mean "not read yet": with origin present that first call answers true, so the reads go on.
- **F14** (card 10): `EndedTeardowns` runs in its own `runRule` after `OriginJudge`'s has returned, with a nil lock. Its local-part and snapshot-tag reads are that rule's `PhasePreLock` reads.

## Self-check

- fix-consistency, MEDIUM: the `plantRunTransition` move edited another concern's file. **Fixed differently**: the helper moves to its own shared helper file instead of `codetipsettle_integration_test.go`.
- fix-consistency, LOW: the overview's sequential-seam decision omitted cards 7 and 8. **Fixed as proposed**: Applies to is now cards 4, 7, 8, 9 and 11.
- fix-consistency, LOW: card 3's adapter-test bullet was stale and the `Recorded` adapter went unchecked. **Fixed as proposed**:
  - The bullet now names card 11's run-lock check.
  - The `Recorded` adapter's coverage is card 2's `TestStatusCodec` and card 12's `TestReconcileOrigin_EndedRuns`.
- fix-consistency, LOW: the `codetip_integration_test.go` header went stale. **Fixed as proposed**: card 12 says the header sentence goes.
- fix-claims, LOW: card 3 gave the wrong reason for the `export_test.go` hit. **Fixed as proposed**: the reason is now the stubbed detached spawn, `swapPushRecorder`.
- fix-claims, LOW: card 10's "as today" overstated the decode error. **Fixed as proposed**: the card now states the codec's error text and the wider parse.
- fix-affected-set, MEDIUM: the landing hand-over's `ReadUpstreamTakeover` call was unnamed. **Fixed as proposed**:
  - Card 4 names `primePushTakeovers` and `takeoverStop` as covered by the push verbs.
  - It leaves `internal/loomcli`'s `runHandOver` to part 4 with the handover readers.
- fix-affected-set, LOW: `newFailureFixtureAt`, `failedPushesOf` and the heddle replayer were missing. **Fixed as proposed**:
  - Added `failedPushesOf`.
  - `failureFixture` registers both replayers. `heddleengine` imports no `boardengine`, so there is no cycle.
  - `pairFixture` builds on `newFailureFixture`, so no `newFailureFixtureAt` is needed.
- fix-proposal, MEDIUM: under F13's wording, `RemoteOnly(true, nil, branch)` would answer false. **Fixed as proposed**: nil tips with origin present answer true.
- fix-proposal, LOW: the planted status path could differ from the tree path the reader reads. **Fixed as proposed**: the status is planted at the `TreeRunFileRels(slug)` status path. `StatusFile` goes through `ResolveRunID` (`internal/shedrun/resolve.go:23-25`).

Every fork returned a report.

## Disputed

None.

## Deferred

None.

## Rulings

- **F2**: the ruling asked that a protected refusal keep today's rejection path. Carried out in card 7:
  - A protected first push runs the `SeamRejectionRetry` call.
  - A protected retry outcome stops the loop as a not-retryable answer.
  - The verdict reads `failRejection`'s judgment facts.
  - `protectedParentWayForward` stays on an `other` record of a push that carried landings.
  - Added the two `TestPushSide` rows.
  - The parent was not told; the ruling holds against `pushclassified.go:327-372` and `:630-690`.
- **F4**: the ruling chose option (a). Carried out:
  - `coalescePushSides` (card 7) and `PushPairAnchored` (card 8) run `refuseUnsuppliedSeams(fabriccore.SeamRunRecordPaths)` before any lock, and card 4's sentence names exactly the verbs that run it.
  - The handover readers' refusal, and the landing hand-over's, are left to part 4.
  - Each of cards 7 and 8 adds a `TestInstallRequiredSeams` row for its verb's refusal.
  - The parent was not told.
- **F11**: the ruling chose option (a). Carried out:
  - Card 2 adds `StatusRecorded`, and its test is renamed `TestStatusCodec` to cover both verdicts. Card 4's reference to the test is updated.
  - Card 3 adds `StatusCodec.Recorded` and its adapter.
  - Cards 10 and 11 read each status file's bytes at `StatusFile` and ask `Recorded` in place of `readRunStatusRows`. That function stays for its `replay.go` callers.
  - Card 9's pre-lock refusal also refuses an unset `SeamStatusCodec` in every scope, since every reconcile runs `recordedRuns` and `recordedPairs` (`originreconcile.go:563-567`).
  - The parent was not told.
