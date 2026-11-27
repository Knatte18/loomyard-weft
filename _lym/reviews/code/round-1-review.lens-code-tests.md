```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, I looked for the test row that would fail if it broke.

**Per behaviour, the pinning test:**
- **`reedengine.NewOver` with a zero `Outside`, through `New`:** pinned. Every existing reedengine test built through `New` (unit and real-tmux integration) runs this path.
- **`Outside.Tmux` handling (told-tmux hook, `floorLine`, spawn branch without server flags), `lifecycle.go:469-479`:** pinned by `TestRecordings_Replay`. The recorded `new-session` call must match the args exactly, or the call is unanswered and `PlaySession`'s cleanup fails.
- **`Outside.CapMemory` through `e.capMemory`, `panememory.go:46`:** pinned by `TestRecordings_Replay`, which holds a recorded `cap-memory` call.
- **`Outside.NewGUID` through `e.newGUID`, `strand.go:387`:** pinned by `TestRecordings_Replay`, whose readings name `guid-1`.
- **`PaneAge` seam and the `PaneProcessAge` export:** pinned by `TestRecordings_Replay` (recorded `pane-age` calls) and by `TestProcessAge/PlatformProbe`.
- **`SessionLiveness` and `SessionRegistrar` in reedengine:** pinned by `TestSessionReads`, moved from loomcli. The new `reedengine:` error prefix is not asserted, which is acceptable.
- **loomcli wiring to the moved reads:** pinned indirectly by the landingshed rows, which use the same functions.
- **`Replay.answer` rules (forward match passing reads, the stale-read fallback, refusing an out-of-order mutation):** only the happy path is covered, by `TestRecordings_Replay`. No row drives a reordered, skipped or unrecorded call. Finding 2.
- **Unanswered-call failure (`PlaySession` cleanup and `Replay.Unanswered`):** none. Finding 2.
- **`Recorder` placeholding (`formatPIDs`, `pidPlaceholder`, `Bindings.placehold`, `placeholdCall`):** pinned only by `TestRecordings_RealTmux` at tier2. Its readings compare would not catch a broken placeholder, since readings hold no pids or paths; only a later tier0 replay of a rewritten recording would. Finding 3.
- **`Bindings.bind` and `replayPID`:** pinned by `TestRecordings_Replay`, since bound args must equal the calls reed makes in the test's directories.
- **`Embedded`, `Load`, `Save`, `Path`:** `Embedded` and `Load` are pinned (tier0 and tier2). `Save` runs only under `-tmuxrec.write`, which is acceptable for a recorder tool.
- **landingshed session world (counting wrapper, `seedLive`, block helpers):**
  - `starts` and `stops` counts: pinned by the `wantStarts` and `wantStops` of every row.
  - `blockStarts`: pinned by `ANonTransientStartFailureHaltsNamingItAndLoomResume` and `AFailedResumeAfterAPauseHaltsNamingItAndLoomResume` (reason contains "permission denied").
  - `blockRegistration`: pinned by `AFailedRegistrationKeepsWaiting`.
  - `blockStatus`: pinned by the `TestPRReview_Wait` row at `prreview_wait_test.go:538`.
  - Pane-death recording: pinned by `ASessionNoLongerLiveWhileTheReviewIsOpenHaltsNamingLoomResume` (halt reason).
  - Not-started recording: pinned by `ATransientStartFailureIsAnError`.
  - Resumed-session liveness: pinned by `ATreeDirtiedAtTheStopIsJudgedOnTheSecondReadAndTheSessionResumed`, which gains a `secondGUID` live check.
- **Registration of the started session's pane process:** loosened. Finding 1.
- **The waitWorld seed moving from always to only when the record names `firstGUID`:** pinned by the `TestPRReview_Wait` rows that reuse a live strand.

**Findings:**

1. **MEDIUM, consistency.** `internal/landingshed/prreview_session_test.go`, first `TestPRReview_Session` row's `check` (diff 001, lines 239-243).
   - Summary: the registration assertion is looser than at the base, and no commit carries a `Loosened-Check:` trailer.
   - At the base it was `cmp.Diff([]string{"fake-guid-1"}, s.registered)`, an exact identity. Now it is `len(s.registered) != 1`, so registering any pid passes, including another strand's pane or the server's.
   - The fix is to assert the exact replayed pane pid. For this recording that is `1&lt;&lt;30 + 4`, from the pane's `{pid:4}` placeholder; it is a literal from the recording, not a derived value.
   - Evidence: diff 001 lines 239-243; `git log` over the range shows no `Loosened-Check:` trailer; session-start-stop.json line 961 shows the strand's pane as `{pid:4}`.

2. **MEDIUM, design.** `internal/testkit/tmuxrec/tmuxrec.go`, `Replay.answer` (lines 993-1016 of diff 004) and `PlaySession`'s cleanup (`session.go`, diff 004 lines 635-639).
   - Summary: no test covers the replay's failure behaviour.
   - `TestRecordings_Replay` replays only scenarios that match their recordings. No row checks that a mutating call that is out of order, changed or absent is refused, or that a read is answered only from state at least as new as the last change. No row checks that an unanswered call is reported, either.
   - A regression that answered any call from any recorded match would keep every current test green.
   - The stand-in rule's "a call with no recorded answer fails" is the kit's central contract and goes unpinned.
   - Evidence: `recordings_test.go` (diff 004 lines 389-407) is the only tier0 test in the package; my grep found no test calling `Unanswered` or `NewReplay`.

3. **LOW, design.** `internal/testkit/tmuxrec/record.go`, `formatPIDs`, `pidPlaceholder`, `Bindings.placehold` and `placeholdCall` (diff 004 lines 72-104, 185-222, 849-863).
   - Summary: placeholding has no tier0 coverage.
   - It runs only inside the tier2 `TestRecordings_RealTmux`, whose comparison is of `Readings`. Readings hold no pids or paths, so a placeholder bug slips that compare. It surfaces only when a rewritten recording is next replayed, or it leaks a machine value into a committed recording unnoticed.
   - A tier0 table over sample formats and outputs would pin it. Example cases: a `#{pane_pid}` field, a server `#{pid}`, and a nested bound path.
   - Evidence: diff 004 lines 469-471 (only `Readings` compared); `StrandReading` fields at lines 792-797.

**Tool calls:**
- One grep for tests referencing `NewOver`, `toldTmux`, `Unanswered`, `NewReplay` or placeholder helpers, plus `git log` for a `Loosened-Check` trailer. My context did not show whether any test outside the diff covers these, or what the trailers of the range carried.
