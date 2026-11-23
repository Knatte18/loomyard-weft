---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/loomcli/gotohook_integration_test.go:54
    summary: The new row's expected error substring is the shared prReviewClearFailure prefix, which a stepStopRunner config or geometry load failure yields too, so the row passes without the hook building a runner or reaching Runner.StopStrand, and its name says the hook stops the strand while it pins a failed stop.
    fix: "In internal/loomcli/gotohook_integration_test.go, change the PRReviewWithARecordedStrandStopsItThroughARunnerOfItsOwn row's wantErr to a substring only the built runner's stop yields, \"could not stop the recorded PR-Review session: shuttle: stop strand guid-1: probe reed before removing it\" (the wording of removeStrandIfPresent in internal/shuttleengine/stop.go). Rename the row so it names the failed stop it pins, for example PRReviewWithARecordedStrandOnAnUnwiredReceiverReturnsTheStopError, and keep the doc comment's lines on that case in step with the new name."
---

### [MEDIUM] The new row cannot tell a built runner's failed stop from a runner that was never built

**Location:** `internal/loomcli/gotohook_integration_test.go:54`

**Issue:** `prReviewGotoAdmitted`'s stop closure (`internal/loomcli/prreviewgoto.go`) fails in two ways.
It returns `load the shuttle and reed config to stop the session: …` when `stepStopRunner` fails,
and the error of `runner.StopStrand(guid)` otherwise.
`clearPRReviewOnGoto` wraps both through `prReviewClearFailure("stop the recorded PR-Review session", err)`,
so the row's `wantErr`, `could not stop the recorded PR-Review session: `, matches either.
The doc comment (lines 20–22) and the commit message say the hook builds its own runner and the stop fails for lack of a reed session.
The assertion admits the case where no runner was built, so it does not pin that claim.
The row does pin the original finding's hazard: a nil `*Runner` would panic at `r.toldErr` (`internal/shuttleengine/stop.go:151`).
The row's name says the hook "StopsIt", yet the row asserts a failed stop.
Equivalent findings from code-correctness (LOW) and code-affected-set (an out-of-lens note) are merged here.
It stays MEDIUM because this row is the one change the task makes, and the assertion falls short of what that row claims to pin.

## Rejected

- No other lens raised a finding.
  Holistically, the diff stays inside the added entry's one-row scope, and the table rework, `reflect.DeepEqual` and the `Fatalf`-to-`Errorf` change only tighten the existing rows.
  The stop failure is deterministic under `tmuxkit.Main`'s private socket (`requireSessionLocked` finds no session), so it fits tier 1.
  No `Root-Cause:` trailer is owed under the "No bug fixed" entry.
  The summary's named tests exist, and its `two-way` door holds for a test-only diff.
