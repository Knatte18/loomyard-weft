---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/loomcli/circling.go:85
    summary: circling accept or continue on a malformed escalation record refuses with the bare parse error, no way forward and no spec row
  - id: F2
    severity: BLOCKING
    class: design
    location: internal/loomcli/decision.go:82
    summary: decision add refuses with a bare error on a status read failure and on any other append failure, naming no retry although the loom raw I/O row promises one
  - id: F3
    severity: LOW
    class: consistency
    location: internal/shedadapters/bouncer.go:688
    summary: the escalation-unreadable Stuck Reason omits the way forward label its sibling Reasons carry, and its test does not assert the clause
  - id: F4
    severity: LOW
    class: consistency
    location: contracts/specs/refusal-spec.md:148
    summary: the config review-key row's Way forward cell does not quote the message's wording
---

Range: no `_lyx/loom/rework/` round exists, so the range is `git diff $(git merge-base main HEAD)..HEAD` with `product.parent` = `main`.
The focus sites were each reached.
The commit-failure refusal and the missing-heading refusal in `decision.go` each have their own clause, spec row and asserting test (`decision_test.go`), which is not a defect.
The unreadable-decision and already-settled-accept Stuck Reasons in `bouncer.go` each name `lyx loom start`, which re-enters the run: `settledGeneration` makes the next entry archive a settled accept, and a malformed decision is re-read on each re-entry.
Each has a shed spec row and an asserting test, which is not a defect.
The sweep for sibling sites reaching a refusal through a shared wrapper found F1 and F2.

### [BLOCKING] Malformed escalation record refuses without a way forward

**Location:** `internal/loomcli/circling.go:85`, reached from `internal/shedadapters/circling.go` `RecordCirclingDecision` via `readEscalation`.
**Issue:** `RecordCirclingDecision` returns `readEscalation`'s error for a malformed `round-<N>-escalation.md` ("bouncer: escalation frontmatter is not valid YAML", round mismatch, bad cause).
`circlingVerb` forwards it through `refuse("%s", err)`.
The refusal is new in this diff, is not raw I/O, names no way forward and has no row in the loom section of `refusal-spec.md`, which breaks `PATTERN-refusal-way-forward`.
The Bouncer-side twin ("escalation record unreadable") has a row; this verb-side route does not.
**Fix:** Map a malformed record to its own refusal naming the file and the way forward (fix or delete the escalation file, then re-run the verb, or `lyx loom start` to re-escalate), add its loom spec row, and add a test in `circling_test.go` that reaches it and asserts the clause.
Use a sentinel error from `readEscalation` so the verb distinguishes a malformed file from I/O.

### [BLOCKING] decision add refusals with no retry clause

**Location:** `internal/loomcli/decision.go:82` and `internal/loomcli/decision.go:102`.
**Issue:** `refuse("read the run status: %s", err)` and `refuse("%s", err)` (the append's I/O errors and a `Validate` error after the bytes are restored) carry no way forward.
The loom section's raw I/O row classes them transient with the way forward "re-run the refused verb", and the Classes table requires a transient message to name the retry.
**Fix:** End both messages with `way forward: re-run lyx loom decision add`, and cover each with a test asserting the clause.

### [LOW] Escalation-unreadable Reason lacks the label

**Location:** `internal/shedadapters/bouncer.go:688`, `internal/shedadapters/bouncer_escalation_test.go:270`.
**Issue:** The Reason names `lyx loom start` but not as a trailing `way forward:` clause, unlike the decision-unreadable and already-settled Reasons.
The test asserts the verb but not the label.
**Fix:** Phrase it as `...; way forward: fix or delete <path>, then run `lyx loom start` to re-escalate the round`, update the spec cell to quote it, and assert `way forward:` in the test.

### [LOW] Config row cell disagrees with the message

**Location:** `contracts/specs/refusal-spec.md:148`, `internal/loomengine/config.go:433`.
**Issue:** The cell reads "set the named key to a positive integer in `loom.yaml`, then re-run the verb"; the message reads "set it to a positive integer in loom.yaml".
The spec says the cell quotes the message so the two cannot disagree.
**Fix:** Align the cell with the message's wording (or the reverse) and keep the test's asserted substring in step.
