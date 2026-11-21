---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/landingshed/finalize_pr.go:504
    summary: The new merge-state read failure path has no behavior coverage.
    fix: Add transient and nontransient post-refusal Status failure rows to TestFinalize_PullRequest in finalize_pr_test.go. Script an initial open status, RefusalOther from Merge, and then the read failure. Assert the read-action text, error versus halting Stuck classification, and no subsequent merge or parent landing.
    origin: lens:code-tests
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/landingshed/finalize_pr.go:596
    summary: An exhausted UNKNOWN poll still prescribes approval or check/conflict repair without evidence of those blockers.
    fix: In mergeRefusalReason, distinguish an open non-draft pull request at the expected head whose only cause is uncomputed mergeability. Give that pending timeout a wait-and-resume recovery without approval, check/conflict repair or cancellation advice. Preserve definite blocker handling. Update the existing never-computed row to assert the pending recovery and reject the unsupported advice.
    origin: lens:code-correctness
  - id: F3
    severity: LOW
    class: consistency
    location: internal/landingshed/doc.go:640
    summary: The new same-minute retry documentation does not describe the composed retry windows.
    fix: Correct the new retry wording in landingshed/doc.go and the mergeOnGitHub comment in finalize_pr.go. State that the outer deadline controls scheduling another backoff, a scheduled sleep can cross it, and each merge-state read has its own bounded poll. Do not claim that all attempts finish within that minute.
    origin: lens:code-affected-set
  - id: F4
    severity: LOW
    class: consistency
    location: internal/landingshed/finalize_pr.go:579
    summary: The resume-only return contradicts the unconditional beam status-command documentation.
    fix: Qualify the beam-refusal descriptions at landingshed/doc.go:634 and finalize_pr.go:571 with the mergeable-state exception, which names resume alone. Retain the intended resume-only return.
    origin: lens:code-affected-set
---

### [BLOCKING] Cover failures in the new post-refusal read

**Location:** internal/landingshed/finalize_pr.go:503–505

**Issue:** The changed refusal rows all read status successfully after the merge refusal.
Existing status-error rows stop before the merge attempt;
the transient merge-error row fails the merge itself.
None reaches this new error return or pins its action and absence of further landing side effects.
PATTERN-test-speed requires coverage of added behavior through the public surface.
The exploration directive assigns BLOCKING to a violation of a listed constraint.
This is a coverage finding, not a claim that the error-handling implementation is wrong.

### [MEDIUM] Keep an uncomputed state distinct from a definite blocker

**Location:** internal/landingshed/finalize_pr.go:577–596

**Issue:** An open pull request at the expected head can remain unknown throughout the bounded poll.
Its sole reported cause is that GitHub has not computed mergeability.
Because MergeableNow is false, the task halt still instructs another person to approve or settle a check/conflict.
It offers cancellation as the alternative.
This leaves the unsupported remedy from the original bug in the timeout case.
Stopping a bounded poll is reasonable; presenting that timeout as evidence of those blockers is not.

### [LOW] Describe the actual retry timing

**Location:** internal/landingshed/doc.go:640–641

**Issue:** With immediate clean reads and repeated refusals, attempts occur at 0, 1, 3, 7, 15, 31 and 63 seconds.
The changed test explicitly expects that last attempt.
If its read becomes unknown, the nested reader can then poll to second 123.
The loop remains bounded, but its new same-minute description is false.
The board entry asks for a bounded wait and does not prescribe a strict one-minute total.

### [LOW] Document the beam exception

**Location:** internal/landingshed/finalize_pr.go:579

**Issue:** The mergeable-state return runs before the beam branch and names resume alone.
The package documentation and function comment still promise a status command for every beam refusal.
The intended new behavior needs that exception stated in both descriptions.

## Rejected

- Code-tests finding 2, as a separate coverage finding: existing rows pin the outer retries and inner timeout.
  A composed timing example substantiates F3, but the goal supplies no stricter total deadline to assert independently.
- An unbounded retry finding: each poll is bounded and the outer deadline is checked after every refusal/read.
  The waits compose; they do not reset the outer deadline.
- An error-shadowing finding: the inner read error is passed directly to githubFailure and returned immediately.

## Review evidence

Reviewed the supplied committed diff and range messages against the decision record and current board entry.
The Root-Cause trailer names the unconditional RefusalOther halt that the diff changes.
The expected-head rename leaves beam consumers compatible; independent beamHead fixture names remain valid.
No PATTERN changes were declared, and the applicable background files require no additional Finalize document.
The three new behavioral rows use literal expectations and an injected clock, with no production waits.

The focused command was `lym gate test --tier 0 ./internal/landingshed ./internal/pullrequest ./internal/beamcli`.
It exited before running tests because the sandbox could not write the hub's gate-holder record.
The error named a read-only filesystem under the hub's `_bolt/.lym/gate` directory.
No fresh test pass is claimed; this environment failure is not a finding against the diff.
