# Fixer report — code review round 1

## Changes

- F1: `internal/fabricengine/doc.go` now says a code-side rejected push is retried over the push backoff by `repushOverOriginContained`, only while origin's tip is contained in the local tip. Fixed as proposed.
- F2: `PushWarpRebaseFreeAt` doc in `spawn.go` says a rejection is retried over the backoff while origin holds no commit the branch lacks. Fixed as proposed.
- F3: `PushRecorded` doc in `bolt.go` says the replay retry runs again over the backoff and a rejection that outlasts it is recorded. Fixed as proposed.
- F4: the code-side "keeps declining" step in `pushanchored_integration_test.go` and the bolt "retry also fails" case in `boltpullfirst_integration_test.go` now count backoff sleeps (at least `len(gitrepo.PushBackoffDelays())`) and assert one `not_fast_forward` record.
  The mail assertion is not on those two: they run in parallel and the push notifier is process-global.
  Fixed differently: the mail is asserted by a new non-parallel row, `ACodeRetryWhoseEveryPushIsRejectedWritesANotFastForwardRecordAndOneMail`, in `TestPushFailureClasses`, which also counts sleeps and the record.
- F5: `sleeps == 0` added to the anchor-stop subtests and the replay-conflict test in `pushclasses_integration_test.go`.
  Fixed differently: the proposed auth and transport assertions on the existing subtests never reach the retry loop (a first-push auth refusal or transport failure skips `settleRejection`), so they were not kept.
  A new row, `ACodeRejectionWhoseRetryIsRefusedForAuthIsNotRetriedAgain`, pins the auth arm: a bare rejection followed by an auth-refused retry sleeps 0 times and writes one `auth` record.
  The proposed change to reuse existing subtests was not made because it cannot catch the regression it targets.

## Self-check

- fix-affected-set, LOW: `pushanchored.go` doc still said "retries a rejection once". Fixed as proposed (reworded to the backoff).
- fix-affected-set, LOW: destructive-refusal, `ErrPushRetriesExhausted` and transport arms of `rejectionRetryKind` lack a sleep-count assertion. Fixed as proposed for the auth arm (new row, see F5).
  The transport arm is deferred: a retry whose fetch fails is unnamed by design and is retried (a test of it slept three times), so a transport failure inside a retry needs a harness this round does not have.
  The destructive-refusal and `ErrPushRetriesExhausted` arms are deferred for the same reason.
- fix-code-correctness, LOW: the auth and never-pushed-transport `sleeps != 0` assertions pass with the defect in place. Fixed as proposed (removed, replaced by the new auth row).
- fix-proposal, LOW: same as the code-correctness finding above, merged. Fixed as proposed.
- fix-consistency: no findings.

## Disputed

None.

## Deferred

- Sleep-count coverage for the transport, destructive-refusal and `ErrPushRetriesExhausted` arms of `rejectionRetryKind` (part of F5's set): reaching them needs a rejection followed by a retry that fails with that class, and no cheap local way to produce one exists in the existing fixtures (a failed retry fetch is classed unnamed and retried by design).
  The auth arm and the anchor-stop and replay-conflict arms are pinned.

## Verification

`lym gate test --tier 1 ./internal/fabricengine` green after the last change.
