# Plan round 1 — fixer report

## Changes

- **F1** (card 1): `held` is now zero unless the wait slept at least once.
  `Network` tells by wrapping the sleep it hands the `Waiter` in a closure that notes it ran; only then is `held` measured from entry to the wait's return.
  The SetGateSlug row of `TestNetwork` gets a fake `Now` that advances on every call, and its uncovered first call still asserts `held` zero, so an unconditional measurement fails it.
- **F2** (card 5): a conditioned notice is named `<time>-<rand>.when-<state>+<cause>.cnotice`, which an old watcher's `noticeNames` skips.
  `noticeNames` lists both suffixes in arrival order.
  The sentence about an old watcher delivering it as a plain notice now says an old watcher leaves it queued until a watcher of this build delivers or drops it.
  `TestQueueConditionalNotice` asserts the conditioned file's name does not end in `.notice`.
  The Edit list gains `noticeName`, re-signed to take the condition (`func noticeName(now time.Time, when *NoticeCondition) (string, error)`), and `noticeNames`.
  It also gains `QueueNotice`, the caller of `noticeName`, which passes nil: `lyx loom validate-plan` flagged the re-sign as `caller-uncovered` without it.
  The review's fix did not name the re-sign or this caller.
  It now passes.
- **F3**: one row per guard branch.
  Card 4's `TestRunner_SendConditioned` gets a no-input-box row: a `*SessionBusyError`, nothing typed, clock unmoved.
  Card 5's watcher test gets a `sessionStateErr` row: the tick returns the error, types nothing and keeps the notice.
  Card 6's `TestRun_ChecksTheBinaryOnlyAtARestartSafeTick` gets an unreadable-record row, with a directory at `RecordPath`: the check never runs.
  These are rows in tests that already exist, so the test budgets are unchanged.
- **F4** (card 12): the surgical-edit list gains the `pauseengine:` to `tenderengine:` change for `errEntryGone` and `errOwnedElsewhere`.
  A grep of `_test.go` files found no assertion on those texts.
- **F5** (card 11): the Intent says `attachCmd`'s `Long` describes exec on Unix and a child on Windows, and keeps the rest of its text.
- **F6** (card 10): the new "revive: agent outside its own pair" row is dropped.
  The existing caller row (`contracts/specs/refusal-spec.md:439`) names revive in its title, trigger and way forward.
  The "revive: unknown slug" row stays.

## Disputed

None.

## Deferred

None.
