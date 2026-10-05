---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/loomengine/config.go:395
    summary: the new review_circling_checkpoint and review_max_bounces refusal names no way forward, has no refusal-spec row, and its test asserts only the key name
---

### [BLOCKING] New loom.yaml review-key refusal breaks PATTERN-refusal-way-forward

**Location:** `internal/loomengine/config.go` (the `review_circling_checkpoint` / `review_max_bounces` loop in `LoadConfig`), `contracts/specs/refusal-spec.md` loom section, `internal/loomengine/config_test.go` `TestLoadConfig_RejectsReviewKeysBelowOne`.

**Issue:**
The diff adds a loom refusal: `LoadConfig` returns `loom config key %q: must be at least 1, got %d` for either new key.
It is reachable through every loom verb that loads config, so `PATTERN-refusal-way-forward` binds it.
Three parts of the pattern are unmet in the same commit:
the message states the constraint but no action (compare the sibling `parent_review_wait_min` refusal, which names what to set);
`contracts/specs/refusal-spec.md` has no row for it;
the reaching test asserts only that the message contains the key, so a message with no way forward passes.

The focus sites are otherwise settled:
the `escalate` path's unreadable-record Stuck names `lyx loom start` and the file to fix or delete, has a refusal-spec row (`escalation record unreadable`), and `TestBouncer_Escalation_MalformedRecordHaltsStuckNamingTheWayForward` seeds a malformed record and asserts the halt and Reason, so it is not a defect.
The other Stuck and refusal paths in `internal/shedadapters` that the diff touches (`circling decision ... is unreadable`, `circling accept ... is already settled`) are context lines in the diff, pre-existing, and not this round's subject.
`loomrecipe.checkReviewBudget` names `review_max_bounces in loom.yaml` and is guarded upstream by the config check, so it is not a verb-reachable refusal.

**Fix:**
Extend the message with the way forward (set the key to a positive integer in `loom.yaml`), add a refusal-spec row for it in the loom section, and make the test assert the message names that action.

## Focus departures

None.
