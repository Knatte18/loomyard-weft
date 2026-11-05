---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: internal/shuttleengine/readiness.go:129
    summary: readinessState's fallback is pinned only through the start path; Runner.SessionIdle and a stale unmatched turn start before the offset have no test row
    fix: In the shuttleengine test file that drives sessionReadiness or Runner.SessionIdle (session_test.go), add rows to the existing table. Row 1 has a non-zero PromptOffset, an events file holding a session start, turn start and turn end before the offset and nothing after, and expects a ready probe. Row 2 has a non-zero offset, a turn start before it with no turn end and nothing after, and expects the held result the hold gives. Row 3 has a zero offset and an empty events file, and expects an unknown reading that takes the pane fallback.
    origin: lens:code-tests
---

### [MEDIUM] readinessState fallback has no test beyond the start path

**Location:** internal/shuttleengine/readiness.go:129
**Issue:** The fallback to the whole-file fold is pinned by the two hook-state start rows (`startup_test.go:1597` and the color-typing row), which fail with `ErrSessionBusy` if it is removed.
`Runner.SessionIdle` reaches the same fallback through `sessionReadiness` (`session.go:266`), and the diff changes its result and documents that in `doc.go`, yet no row drives it with a non-zero offset and no later signal.
The stale case the focus names, an unmatched turn start before the offset, is also unpinned.
Reading the code, that case is handled: `unmatchedTurnStart` (`send.go:407`) already parses the whole events file, so the fallback fold and the hold read the same history and the existing hold releases it.
The missing row is what would keep that so.

## Rejected

- Refold also on cause `no-state` (code-correctness F1): a signal past the offset is activity after the load turns, so the pane fallback is the intended reading there; the lens did not show a parser signal that produces it on the failing path.
- Other readers (`SessionState`, `conditionedReading`, `continueAPIErrorStall`) stay offset-bounded (code-correctness F2): they judge what the run did since the prompt, as `doc.go` states for them, and `activity.go:243` already has the same split.
- Rows for the zero-offset guard and the non-`no-signal` cause guard as no-ops (code-tests F1, partly): a dropped guard changes no result, only a repeated read, so no behavior is unpinned; the zero-offset and unknown rows in F1 stay as a pin of the fallback to the pane.
- Stale-state risk in `readinessState` (focus 1): not a defect, since the hold already folds the whole file.
- Unmade asks, a bounded re-poll and a retry inside the seat start (focus 3): the send already re-polls for `send_ready_timeout_s`, and the entry says "consider" a retry; the summary records why it was left out, so no ask is unmade.
- `fakes_test.go` change weakening a pane-fallback assertion (focus 5): only the hook-state start rows use the fake, and no pane-fallback assertion moved.
- `doc.go` sentence (focus 6): it matches `readinessState` on the no-signal cause and the non-zero offset, and sits in the session-state folding paragraph.
