```yaml
lens: code-tests
outcome: reported
```

Task: for each behaviour the diff adds or changes, name the test that pins it, or none.

**Per behaviour, the pinning test**

- **`wait` and `pause` early wake:** `TestPRReview_Wait`, row `AChangeToTheRecordEndsTheSleepEarlyAndTheContinueFlagClosesThePullRequestAfterTheFloor` (`prreview_wait_test.go`). It would fail if a change did not cut the sleep. Its 10-second guard fails if the sleep is never cancelled, and the final `Done` outcome with `continueClose` shows the flag was acted on.
- **`pause` floor after a change:** the same row. The `[]time.Duration{time.Second, time.Second}` assertion on `w.sleeps` pins the remaining floor, but only for the zero-elapsed case. The case where part of the floor has already passed, so the rest is shorter, has no row. Neither does the case where the floor has fully elapsed and there is no second sleep.
- **`pause` when the timer fires first:** the existing wait rows, which now go through `pause` with `changes == nil`.
- **`wait` ends the watch:** the same new row asserts `watchStops == 1`. Only the changes-delivered path is checked; the other return paths (merged, closed, cancelled) are not.
- **Context cancellation during `pause`:** the existing cancellation rows cover the timer branch. Nothing covers cancellation in the changes branch or during the floor sleep.
- **`checkWiring` requires `RecordChanges`:** `prreview_test.go:364`, the nil-seam table row.
- **`wake` sets `wokeAt`:** only indirectly, through the new row. The row's clock stands still, so it cannot tell `wokeAt` apart from a zero time or a stale one.
- **`wait` with a nil channel:** the existing rows via `prReviewTestDeps` and `newWaitWorld`.
- **Seam-table entries (`wire_seams_test.go`, `wiring_seams_test.go`, `envkit`):** these are the pins themselves.
- **`pullRequestDeps` wiring of `RecordChanges` to the `pull-request.json` path under `SelfRunID`:** none. The grep for `pullRequestDeps(` and `RecordChanges` in the loomcli tests found only the seam table.
- **`watchFileChanges` in `landingdeps.go`:** none. These behaviours are uncovered:
  - an event reaches the channel;
  - bursts merge into one value;
  - an unwatchable directory gives a nil channel plus the Warn;
  - `stop` closes the watcher.
- **`openLandRequestWatch` now calling `watchFileChanges`:** none. The grep of all `*.go` found no test reference to `openLandRequestWatch` or `watchFileChanges`. A Beam-Await wake from a land-request change is not exercised through this path. Its changed Warn text and the dropped directory-versus-file filter are likewise unpinned.

**Findings**

1. MEDIUM, design, `internal/loomcli/landingdeps.go:371-396`. `watchFileChanges` is new production code with no test, and it is the only real fswatch consumer of the PR-Review wake. A test with a temp dir would write the file, expect one channel value, expect merged bursts, expect a nil channel for a missing dir, and expect `stop` to close the watcher. No other test would fail if the file filter or the event forwarding broke. Evidence: the grep over `*_test.go` found no hit; `landingdeps.go:371-396`.
2. LOW, design, `internal/loomcli/landingdeps.go:365-367`. The `pullRequestDeps` wiring of `RecordChanges` is unpinned. If it watched the wrong path, no test would fail, and the fake seam in `landingshed` cannot catch that. This could share the fix for finding 1 if the test asserts the watched path. Evidence: the seam table lists the key only as intentionally nil.
3. LOW, design, `internal/landingshed/prreview_wait_test.go:512-547`. `pause` has only the zero-elapsed floor case. A row with elapsed time under the floor, and one with elapsed time over it (no second sleep), would pin the `rest > 0` branch. These can be rows in the same table. Evidence: the comment at 540 says the clock stands still.
4. LOW, scope, `internal/loomcli/pawlwiring.go:146-157`. The Pawl watch was refactored onto the shared helper with no test pinning that Beam-Await still gets the channel and the nil fallback. This overlaps with finding 1 and is cleared by the same test.

**Tool calls**
1. `grep` over loomcli and landingshed `*_test.go` for `RecordChanges`, `pullRequestDeps(` and `fswatch`: my context did not show whether any test references the new wiring or the watcher.
