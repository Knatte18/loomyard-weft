---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: internal/shedadapters/bouncer.go:700
    summary: A failed writeEscalation halts the run Failed with the forwarded write error and no way forward, no shed spec row and no reaching test
  - id: F2
    severity: LOW
    class: consistency
    location: contracts/specs/refusal-spec.md:121
    summary: The escalation-record and circling-decision unreadable rows say malformed, but the Stuck Reason also fires on a read I/O failure and on an unreadable parent notice file
---

Range: no `_lyx/loom/rework/` round exists, so the range is `git diff $(git merge-base main HEAD)..HEAD` with `product.parent` = `main`.
`gofmt -l` over every Go file the diff touches lists nothing; `go vet` and `go test` over `loomcli`, `shedadapters`, `shedengine` and `discussionparser` pass.

Focus sweep 1, `loomcli/circling.go` and `loomcli/decision.go`.
Every `refuse(` site in both files carries a `way forward:` (the `circlingWay*` and `decisionWay*` constants, or the inline text of the `--by`, empty-flag and commit-failed sites).
The forwarded helper errors (status read, record I/O, append, recipe lookup) all end in a retry or a named fix.
Each site has a loom-section row in `refusal-spec.md` and a case in `TestCirclingVerb_Refusals`, `TestDecisionVerb_Refusals` or `TestDecisionVerb_CommitFailureNamesWayForward` that asserts `way forward:`.
The slug and unknown-slug refusals come from `resolveReviewTarget`, which names its way forward and has rows.
No finding.

Focus sweep 2, `internal/shedadapters`.
The three Stuck Reasons and degrade messages the diff adds (unreadable circling decision, already-settled accept, unreadable escalation record) each name a way forward, have a shed-section row, and are asserted by `bouncer_circling_test.go:189,203` and `bouncer_escalation_test.go:270`.
The escalation render failure only warns and degrades to the plain Reason, so it is not a refusal.
The sweep found one site that forwards a shared-helper error without a way forward (F1) and one imprecise row (F2).

### [MEDIUM] Failed escalation write names no way forward
**Location:** `internal/shedadapters/bouncer.go:700`, `internal/shedadapters/escalation.go:52,59`
**Issue:** `escalate` returns `writeEscalation`'s error as the producer error, so Shed persists the run `failed` with `shedadapters: <name> (bouncer): write escalation <path>: <err>`.
The message carries no way forward, the shed section has no row for it (its "raw I/O" row covers status, seed and lock files only), and no test makes `writeEscalation` fail through `escalate`.
This is a new halt the diff introduces on the path every escalation takes.
`PATTERN-refusal-way-forward` requires each refusal in shed to name its way forward and land its row and reaching test in the same commit.
The pre-existing approve and commit error returns sit outside this diff.
**Fix:** Append a way forward to the error, such as re-running `lyx loom start` once the named path is writable, since a re-step re-enters `escalate` and rewrites the record.
Add a shed-section row (transient, same way forward) and a test in `bouncer_escalation_test.go` that makes the write fail and asserts `way forward:` and the path.

### [LOW] Unreadable rows understate their trigger
**Location:** `contracts/specs/refusal-spec.md:121`, `contracts/specs/refusal-spec.md:122`
**Issue:** The Stuck Reason fires on any `readEscalation` or `readCirclingDecision` error, including a plain read I/O failure and a failed read of the parent notice file (`escalation.go:78,100`).
The rows name only a malformed record, and the Reason says "fix or delete" the escalation file even when the failing file is the parent notice.
**Fix:** Widen both triggers to "unreadable or malformed", and have the Reason name the file the error names, which `%v` already carries, so the row and the message agree.
