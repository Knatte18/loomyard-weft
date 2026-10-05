---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: internal/loomcli/decision.go:112
    summary: decision add commit-failure refusal names a way forward and has a test, but refusal-spec.md carries no row for it
  - id: F2
    severity: BLOCKING
    class: scope
    location: internal/loomcli/decision.go:104
    summary: decision add refuses a record with no Decisions heading through refuse of the bare error, with no way forward, no spec row and no reaching test
  - id: F3
    severity: BLOCKING
    class: scope
    location: internal/shedadapters/bouncer.go:604
    summary: Bouncer halts Stuck on a malformed or already-settled circling decision file with no way forward, no spec row and no way-forward assertion
---

Range: `status.json` gives `product.parent` = `main`, no `_lyx/loom/rework/` exists, so the range is `git diff $(git merge-base main HEAD)..HEAD`.
Focus: every refusal the diff adds or changes in webster, shed or loom.
Sites checked and found compliant: `--by` and empty-flag refusals, Discussion-Write running, no decision record, check finding after append, the circling verbs' refusals, `loom.yaml` review keys below 1, the malformed escalation record.
Not refusals, so out of scope for this pattern: ledger parse failures (every caller degrades or contributes nothing), the unearned-CIRCLING narrowing (a warning), and the `NewBouncer`, `checkReviewBudget` and `newBouncerEntry` guards (wiring guards, unreachable once `loomengine.Load` has validated the keys).

### [BLOCKING] decision add commit failure has no refusal-spec row

**Location:** `internal/loomcli/decision.go:112`, `contracts/specs/refusal-spec.md`
**Issue:** `decisionVerb` refuses with "the entry is appended but the commit failed: ...; way forward: run \"lyx fabric commit\"" and `TestDecisionVerb_CommitFailureNamesWayForward` reaches it, but the loom section has no row for it.
`PATTERN-refusal-way-forward` requires the spec row in the same commit as the refusal.
**Fix:** add a transient row, `decision: commit failed after the append`, whose way forward quotes the message's clause.

### [BLOCKING] decision add refuses a malformed record without a way forward

**Location:** `internal/loomcli/decision.go:104`
**Issue:** `AppendDecision` returns errors for a record with no `## Decisions` heading (`decision record has no "## Decisions" heading`), and `decisionVerb` passes any non-`ErrNotExist` error straight to `refuse("%s", err)`.
The message names no action, there is no spec row, and no test asserts the case.
**Fix:** give the no-heading refusal its own `way forward:` clause (restore the heading in the decision record, then re-run the verb); keep raw I/O errors in the grouped raw I/O row; add a spec row and a test that asserts the clause.

### [BLOCKING] circling decision file refusals in the Bouncer name no way forward

**Location:** `internal/shedadapters/bouncer.go:604`, `internal/shedadapters/bouncer.go:628`
**Issue:** `readCirclingDecision` gained a cause check and the Bouncer halts Stuck with "circling decision for round N is unreadable: ..." and "circling accept for round N is already settled".
Neither names a way forward, the shed section has no row for them (the escalation-record row exists, these do not), and the existing tests assert only the failure text, not a way forward.
**Fix:** append a `way forward:` clause to both Reasons (fix or delete the named decision file, then `lyx loom start`; for an already-settled accept, `lyx loom start` after deleting the settled file is wrong, so name `lyx loom goto --to <segment>` to re-enter it), add one shed spec row each, and extend the tests to assert the clause.
