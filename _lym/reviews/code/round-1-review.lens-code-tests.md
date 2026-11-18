```yaml
lens: code-tests
outcome: reported
```

Restating my task: I'm the code-tests lens. For each behaviour the diff adds or changes, I name the test that would fail if it broke.

**Per behaviour, the test that pins it:**

1. **A records branch origin still holds is pushed again, and the commit made during the push lands.** Pinned by `TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide/PrimeRecordsCommitDuringThePushIsPushedAgain` (`internal/fabricengine/pushanchored_integration_test.go:523`). Without the fix, the first `HasUnpushed` reads true, no deferred record exists, and the old fallback returns "ended on origin", so the `t.Fatalf` on `err != nil` fires. The test also pins the landing: the bare holds the hook's commit and no failed-push record is written. It is not redundant with `PrimePushesRecordsSideOnly` at line 497, which has no commit during the push. The redundancy report marks this function as "not judged", and that is no finding.
2. **A branch gone from origin still reports "ended on origin", without a retry.** Pinned by `AnEntryEndedOnOriginSinceTheLastBoltFetchWritesNoRecord` and `TheDoneMarkArrivingDuringTheBackoffEndsItWithNoRecord` (`internal/fabricengine/pushclasses_integration_test.go:702` and `:726`). Both assert the "ended on origin" text, and the first also asserts `sleeps == 0`. If `branchGoneOnOrigin` were misread as "not gone", the loop would sleep, so the sleep count would catch a retry.
3. **The deferred-record check runs before the origin lookup.** Pinned by the existing `Deferred_*` rows (`pushanchored_integration_test.go:392`). If the order flipped, a deferral would read as `errRecordsUnpushedPastPush`, and the call would end in that error instead of the deferred `*PushFailedError`.
4. **The retry stops once the backoff is spent (`attempt == len(delays)`) and returns an error wrapping `errRecordsUnpushedPastPush`.** No test pins this. A grep for "still on origin", "holds commits its push did not carry" and `errRecordsUnpushedPastPush` finds nothing in fabricengine's tests.
5. **A failed lookup on origin returns the lookup error ("look up branch … on origin after its push"), never the ended message.** No test pins this.
6. **The retry sleeps `gitrepo.PushBackoffDelays()` through `opts.BackoffSleep`.** No test pins this. The new subtest passes a no-op sleep but counts no calls, so code that ignored `opts.BackoffSleep` or slept the wrong delays would still pass, with the test just running about a second slower.
7. **The doc comment changes in `doc.go` and `pushanchored.go`, and the replaced duplicate line in the test header.** These are static prose, so they are exempt.

**Findings:**

- **MEDIUM | design | `internal/fabricengine/pushanchored.go:230`**: no test drives a records branch that stays unpushed on every attempt while origin holds it. Nothing pins that the loop stops after `len(gitrepo.PushBackoffDelays())` sleeps and returns an error wrapping `errRecordsUnpushedPastPush`. An off-by-one, or an unbounded loop if the bound were dropped, would pass the suite. Evidence: the bound is at `pushanchored.go:230`, and the grep of `internal` `*_test.go` for the sentinel's text and name found no hit. Fix: add a row, for example a `post-receive` hook that commits on every push, or a records upstream whose tracking ref never moves. The row asserts `errors.Is(err, errRecordsUnpushedPastPush)` (exposed through `export_test.go`) and a sleep count of `len(gitrepo.PushBackoffDelays())`, using the `countingBackoff` helper `pushclasses_integration_test.go:681` already uses.
- **LOW | design | `internal/fabricengine/pushanchored.go:332-335`**: the new error path for a failed origin lookup has no test. The diff's purpose is to stop a lookup failure from reading as "ended", and nothing would catch that regression. Evidence: `pushanchored.go:333-335`, and the same grep found no test asserting "look up branch". Fix: add a row where `ls-remote` fails for a reason other than transport after the push. One option is a records remote URL rewritten to an unreadable path between the push and the check, though doing that between the two steps may be hard to arrange. If no hermetic fixture can do it, add `//lym:seam`-style coverage at that function.
- **LOW | design | `internal/fabricengine/pushanchored_integration_test.go:539`**: the new subtest passes `BackoffSleep: func(time.Duration) {}` and never counts its calls. So it does not pin that the retry sleeps through `opts.BackoffSleep`, nor how many times it sleeps (it should be once). Evidence: line 539 of the test, against `pushanchored.go:213-216` and `:234`. Fix: replace the no-op with `countingBackoff(&amp;sleeps, nil)` and assert `sleeps == 1`.

Tool calls:
1. Grep of `internal` `*_test.go` for "still on origin", the sentinel's message, the `errRecordsUnpushedPastPush` name and "look up branch". My context showed no search of the tests for coverage of the exhaustion and lookup-failure paths.
