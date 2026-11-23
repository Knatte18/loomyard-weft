```yaml
lens: code-tests
outcome: reported
```

Lens code-tests: for each behaviour the diff adds or changes, the test row that would fail if it broke.

**Per behaviour, the test that pins it:**

1. **A wake that reads no status halts once the session is dead** (`internal/landingshed/prreview.go:706`). Pinned by `TestPRReview_Wait/ASessionThatEndsWhileTheReviewIsOpenHaltsNamingTheResume`.
   - Trace: the entry status is taken as the first wake's poll, and the session is live then. The session dies at sleep 1. Wake 2 is under a minute after the entry read and forces nothing, so it reads no status and halts at line 706.
   - The row checks Stuck, Halt, the reason's two parts and the start and stop counts against literals, so its expected values do not come from the code.
2. **A wake that reads a status and leaves the review open halts once the session is dead** (`prreview.go:735`). No row pins this.
   - This is the mutation survivor. Negate line 735 and the dead-session row still passes: wake 2 returns nil and the wait goes on, and a later wake that reads no status catches the dead session at line 706 instead.
   - Delete the branch and the row also passes.
   - The branch matters in steady state. After about six idle wakes the gap between wakes reaches `prWaitCeiling`, which equals `prPollInterval` (one minute), so every wake reads a status. Line 706 is then never reached, and line 735 is the only halt.
3. **The halt's message and Halt flag** (`prreview.go:759`). Pinned by the same row as item 1; it asserts "PR-Review session of pull request #7 ended" and "lym loom resume".
4. **A live session lets the wait go on** (`prreview.go:756`). Pinned by every existing `TestPRReview_Wait` row that waits on an open pull request with the started guid-1 session and cancels at sleep N, such as the stay-open and rewrite-description rows. If the check were inverted, those rows would halt instead of returning `context.Canceled`.
5. **A failed liveness read during the wait ends the call with an error** (`prreview.go:753-755`). No row pins this.
   - `prreview_session_test.go:489` (`ASessionLiveErrorIsAnError`) sets `liveErr` before the call, so the read fails in `ensureSession` at entry. It never reaches `haltOnEndedSession`.
   - Both sites wrap with the same message, so that row cannot tell them apart.
6. **A wait entered on an ended review passes `keepsSession=false`** (`prreview.go:306`). Not pinned on its own.
   - `AReviewEndedAtEntryKeepsNoSessionSoItsDeadStrandDoesNotHaltTheWait` also passes if line 306 passes true, because the `rec.ReviewEnded` guard at `prreview.go:749` returns nil first.
   - It also passes if that guard is removed, because the flag is false. The row fails only when both are broken.
7. **A pull request merged at entry passes `keepsSession=false`** (`prreview.go:340`). Not observable: the first wake judges the pending merged status and always ends the call, so no test can pin it. No finding.
8. **A pull request closed at entry passes `keepsSession=true`** (`prreview.go:340-346`). No row has the session die while a closed-at-entry review waits, for example on a continue flag over a dirty tree. The branch that would halt is the uncovered line 735 from item 2, so it goes in that finding.
9. **The changed call in `TestPRReview_WaitWithNoLastSetTextTakesTheDescriptionAsTheBaselineAndEditsNothing`** (`prreview_wait_test.go:1131`). It still checks what its name says. Passing true would make the wait read `SessionLive` on a record with no strand and halt, so false is the right argument. No finding.
10. **The doc comment lines** in `doc.go:24` and on `PRReview`, `wait`, `wake` and `prWaitState` are static prose and exempt.

**Findings:**

- **MEDIUM, design**, `internal/landingshed/prreview.go:735`: no test pins the halt on a wake that reads a status. Negating or deleting the `else if end := p.haltOnEndedSession(...)` branch passes every test, because the dead-session row halts at line 706 on a wake that reads none.
  - Once the wait has sat idle long enough, every wake reads a status, so line 735 is the only halt in steady state.
  - Fix: add a row where the session dies after the wait reaches that state, or where the entry status is stale so the wake after the death reads a status. Assert the halt.
  - Evidence: `prreview.go:705-737`, `prreview.go:24-28`, `prreview_wait_test.go:996-1001`, and the mutation report's survivor at `prreview.go:735`.
- **LOW, design**, `internal/landingshed/prreview.go:753-755`: no row reaches the liveness-read failure inside the wait. The only `liveErr` row, `prreview_session_test.go:489`, fails in `ensureSession` at entry, before the wait runs.
  - Fix: add a row that sets `liveErr` after the session starts, for example at sleep 1. Assert that `Call` returns an error naming the liveness read.
  - Evidence: `prreview_session_test.go:87-91`, `prreview_session_test.go:488-489`, `prsession.go:80-83`.
- **LOW, design**, `internal/landingshed/prreview_wait_test.go:1009` (`AReviewEndedAtEntryKeepsNoSessionSoItsDeadStrandDoesNotHaltTheWait`): the row does not pin `keepsSession=false` at `prreview.go:306`, which its name claims. The `rec.ReviewEnded` guard at `prreview.go:749` returns nil before the flag is read, so the row passes with either value.
  - Fix: drop the redundant `rec.ReviewEnded` guard so the flag is what the row pins, or reword the row to say it pins that an ended review is never watched.
  - Evidence: `prreview.go:306`, `prreview.go:749`, `prreview_wait_test.go:1009-1017`.

Tool calls:
- Grepped the test files for `live["guid-1"] = false` and `liveErr`: my context did not show whether any other row kills the session or fails a liveness read during the wait.
