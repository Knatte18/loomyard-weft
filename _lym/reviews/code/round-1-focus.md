---
round: 1
focus:
  - "internal/creelengine/machinemail_test.go, the hour subtest, with the scheduler and machine hour loop it drives: after the reorder, can any pass besides the first (from start itself, a wake queued before it, a hook, or any other timer) still read the store before Advance(time.Hour) and the clock after it, or otherwise arm a wait the advance does not fire?"
  - "The same subtest: with both posts and the hold report ahead of e.start, does it still check what its name states (scheduler started, no take, return at the hour, last reported reason or bridge unsettled) for both messages?"
  - "_lym/landing/summary.md and the new test comment against the code: are the claims about what a post wakes before and after the start, what the first pass arms, and which other subtests still cover a post after the start borne out? Name the covering subtest or find none."
  - "Commit f84f1a937's Root-Cause trailer and message: does the named defect match the one the diff removes, as traced through the loop's code?"
---

The range is one test-only commit that reorders a subtest's setup and adds a comment.
The decision record settles that the fix is test-only and that the red-first check does not apply.
The open questions are whether the race is closed and whether the account of it matches the loop's code.
