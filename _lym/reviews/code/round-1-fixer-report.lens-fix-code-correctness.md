```yaml
lens: fix-code-correctness
outcome: reported
```

Task: I checked the fix-code-correctness lens on the changes to `prreview.go` and `prreview_wait_test.go`. I found one issue.

**Finding 1 — LOW.** The fix leaves `AReviewEndedAtEntryKeepsNoSessionSoItsDeadStrandDoesNotHaltTheWait` unable to pin the flag its name claims (F5).
- Changed text: the F5 guard change in `haltOnEndedSession` (`if !state.keepsSession {`), together with the F1 change (`if !polled { return nil }`).
- Trace of the row: the record has `ReviewEnded` and a dead strand, and `scripts` is a single transient `DeadlineExceeded`. `Call` enters `wait` with `entryStatus` nil, so wake 1 polls and the poll fails transiently, giving `polled == false`. The wake then returns nil at `if !polled`, and `haltOnEndedSession` is never reached. Wake 2 does the same, and the test cancels after sleep 2.
- Effect: the row passes whether `prreview.go:306` passes `false` or `true`. This is the defect F5 reported, now caused by F1 instead of the old `rec.ReviewEnded` guard.
- Why no other row can pin the flag: with `ReviewEnded` set and a status read, `sessionStopWanted` is true, so `stopped` is true and the `else if` is skipped. With a merged status, `judge` ends the call first. The `keepsSession` argument at `prreview.go:306` therefore cannot change behaviour, and the F5 name and the summary's claim about it are unpinned.
- Suggested fix: rename the row to say what it observes (a wait entered on an ended review keeps waiting through a transient poll failure). Or drop the `keepsSession` argument at the two entry sites where no status can reach the check.

Checked and found correct:
- F1 and F3: the row now has `statusScript(open, open)` and advances `now` by `prPollInterval` at `n == 1`. Wake 2 polls, judges open, takes the `else if` after `judge`, and halts. Negating that condition (the surviving mutation) now fails the row.
- F2 and F4: the new row fails with the old code. The old code returned `&ending{err}` on the liveness error, so `Call` would not wrap `context.Canceled`, and `endSession` would add a stop that the `assertSessionCounts(1, 0)` check would catch. With the fix, the failed liveness read at wake 2 logs one Warn and returns nil, and wake 3 logs none because `liveWarned` stays set. The cancel after sleep 3 gives `context.Canceled`, one start and no stop.
- F5 guard removal in the stopped path: a record with `ReviewEnded` always makes `sessionStopWanted` true, so `stopped` is true and the `else if` is skipped. Dropping `|| rec.ReviewEnded` cannot halt a call that has ended its review.
- Callers of `haltOnEndedSession`: the only remaining caller is the `else if` in `wake`. The signature is unchanged and `state.liveWarned` is set and cleared consistently.

Further tool calls: none. My context already held every changed file.
