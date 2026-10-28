---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: internal/fabricengine/doc.go:753
    summary: package doc still says a code-side rejected push is retried once after a fetch, contradicting the updated push classification section
    origin: lens:code-affected-set
    fix: In internal/fabricengine/doc.go lines 753-754, delete the sentence "A code-side rejected push is retried once, after a fetch, only when the fetched remote tip is already contained in local HEAD, so a stale remote-tracking ref recovers by itself while a real divergence does not." or reword it to say a code-side rejection is retried over the push backoff by repushOverOriginContained, only while origin's tip is contained in the local tip, so a stale tracking ref or a lost race recovers while a real divergence is reported at once. Keep the following sentences about the remaining ErrPushRejected.
  - id: F2
    severity: LOW
    class: consistency
    location: internal/fabricengine/spawn.go:265
    summary: PushWarpRebaseFreeAt doc says a rejection is never retried here, but the code side now retries it over the backoff
    origin: lens:code-affected-set
    fix: In the PushWarpRebaseFreeAt doc comment in internal/fabricengine/spawn.go, replace "A rejection is a human-decidable condition, never retried here." with a sentence saying a rejection is retried over the push backoff only while origin holds no commit the branch lacks, and the rejection that remains is a human-decidable condition.
  - id: F3
    severity: LOW
    class: consistency
    location: internal/fabricengine/bolt.go:116
    summary: PushRecorded doc says a rejected push is retried once, but settleRejection reruns the replay retry over the backoff
    origin: lens:code-affected-set
    fix: In the PushRecorded doc comment in internal/fabricengine/bolt.go, replace "is retried once after the board's local commits are replayed onto origin's" with wording saying the replay retry runs again over the push backoff while it fails with a bare rejection or an error no class names, and a rejection that outlasts the backoff is recorded.
  - id: F4
    severity: LOW
    class: scope
    location: internal/fabricengine/pushanchored_integration_test.go:357
    summary: a rejection that persists through every attempt is asserted for one record and one mail on the records side only, not on the code or bolt side
    origin: lens:code-tests
    fix: In the code-side "remote that keeps declining" step of TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide, count the backoff sleeps through a counting BackoffSleep and assert at least len(gitrepo.PushBackoffDelays()) of them, and assert exactly one not_fast_forward failed-push record and one mail afterwards; add the same sleep-count and record assertion to the bolt PushRecorded rejection case in boltpullfirst_integration_test.go:610.
  - id: F5
    severity: LOW
    class: scope
    location: internal/fabricengine/pushclassified.go:200
    summary: no test pins that the loop stops at once, with no backoff sleep, on an anchor stop, replay conflict, auth refusal or transport failure
    origin: lens:code-tests
    fix: In pushclasses_integration_test.go add a counting BackoffSleep to the existing anchor-stop, replay-conflict, auth and transport subtests of TestPushFailureClasses and assert the sleep count is 0 in each, rather than adding new tests.
---

### [MEDIUM] Package doc keeps the old retry-once sentence

**Location:** internal/fabricengine/doc.go:753
**Issue:** The package doc still states a code-side rejected push "is retried once, after a fetch".
The same file's push classification section, about line 929, now describes `repushOverOriginContained` and `settleRejection` retrying over the backoff, so the doc contradicts itself.
The task's scope names the `doc.go` push classification section as stating the new retry, and this sentence is part of that description.

### [LOW] PushWarpRebaseFreeAt doc says a rejection is never retried

**Location:** internal/fabricengine/spawn.go:265
**Issue:** `PushWarpRebaseFreeAt` calls `pushClassified` with a nil retry at `spawn.go:275`, which runs the code-side `repushOverOriginContained` over the backoff.
The doc comment's "never retried here" is now false.

### [LOW] PushRecorded doc says a rejection is retried once

**Location:** internal/fabricengine/bolt.go:116
**Issue:** `PushRecorded` passes `retryBoltPush` to `pushClassified` (`bolt.go:126`), and `settleRejection` reruns it over the backoff, so "retried once" no longer describes it.

### [LOW] Persisting rejection pinned on the records side only

**Location:** internal/fabricengine/pushanchored_integration_test.go:357
**Issue:** The acceptance criterion that a rejection persisting through every attempt writes one `not_fast_forward` record and one mail is asserted for the records side by `ARecordsRetryWhoseSecondPushIsRejectedAgainWritesANotFastForwardRecord`.
The code-side declined-always step asserts only `IsPushRejected`, an unmoved bare and no local movement, and the bolt case only gained a no-op sleep.
The loop is shared, so the gap is in coverage of the side wiring, not of the loop.

### [LOW] Stop-at-once on named failures has no assertion

**Location:** internal/fabricengine/pushclassified.go:200
**Issue:** `rejectionRetryKind` makes anchor stop, replay conflict, destructive refusal, auth refusal, retries exhausted and transport failure non-retryable.
No test counts backoff sleeps for them, so a regression that retried them would pass: the existing subtests would only slow down or end in the same error.

## Rejected

- Code-correctness: lock held for the whole backoff while the weft push lock is held: the decision record's Open risks accepts that a persisting rejection waits out the schedule, and an unreachable origin already waits the same way.
- Code-correctness: code side loses the stale-ref retry on the early `return job.push()` paths: the local-branch mark is refused by `PushRebaseFree` itself and an unreadable branch is a path where the push cannot succeed; the lens itself judged it not a defect.
- Code-tests: no test for an unnamed-error retry or an upper bound on the loop: the loop's lower bound and the joined-rejection shape are pinned, and the bound is the length of `PushBackoffDelays()` read directly in the code; the added test would pin the implementation, not an acceptance criterion.
- Code-tests: no dedicated stale-tracking-ref test after the one-shot retry's removal: the declined-once anchored case and the new Code row cover the retry it folded into.
- Code-tests: no bolt-without-paths way-forward row: bolt and records share `recordsConflictWayForward`, which the records no-paths row already pins.
- Code-tests: bolt side landing on a later attempt has no row: the loop is shared and the Records row exercises `settleRejection` with a replay retry; covered under F4's side-wiring gap.

Holistic: the diff makes every change the amended entry asks for.
The retry covers the three sides, `pushRebaseFreeRetryingOnce` is gone with no reference left, the code-side retry pushes only while origin is contained in the local tip, and nothing forces, rebases or merges.
Record and mail counts for a persisting rejection are unchanged, and the way forward drops "another machine" and the empty `paths:` clause.
`_lym/landing/summary.md` claims match the diff.
