---
round: 1
focus:
  - "Board entry bullet 'Keep a cwd-reset notice out of the failure identity and the note body': the diff has no code that strips or ignores the harness's 'Shell cwd was reset' line (grep of internal finds none). Settle whether a cwd-reset line in a first run's output can still become a failure identity or reach the note body through `flakyFailure.failure.Tail` and `writeFailureOutput`, and whether the diff makes any change for this bullet."
  - "Board entry bullet 'Check how triage derives identities and whether verify-command can stand for a failure of the command': in `internal/websterengine/triage.go`, `flakyFailure.commandFailure` classifies by `FailureKindOpaque`. Settle whether every first run whose output names no failing test lands in that kind, and whether `splitFlaky`, `triageWarnings`, `writeTriageFrictionNote` and `AppendIntegrationTriage` keep command failures out of the flaky-test list on every path."
  - "`VerifyGateNotes.addFlaky` in `internal/websterengine/verifygate.go` dedups by `failure.ID`. Settle whether two opaque failures sharing one ID in a single first run, or the same identity recurring across gate evaluations with different logs, lose output or a log that the entry asks to keep."
  - "`writeFailureOutput` and `writeTriageFrictionNote` in `triage.go`: settle whether a tail containing a code fence, or a tail with trailing newlines, breaks the note's markdown block or its indentation, and whether the empty-output case reads correctly."
  - "Commit range holds a revert and a reapply of the spec commit; the decision record carries a parent entry waiving the red-first gate. Settle that the net diff of `contracts/specs/webster-spec.md` and `internal/websterengine/doc.go` states what the code now does, and that `Flaky()` removal (renamed to the unexported `flakyFailures`) leaves no caller or doc reference behind."
  - "Test economy and loosened checks in `summary_test.go`, `triage_test.go`, `verifygate_test.go`: settle whether any test at the review base had an assertion widened or removed, and whether any new expected value is computed the way the code computes it."
---

Seed focus drawn from the board entry's three 'What to do' bullets and the diff's file list.
The redundancy and mutation reports raise no candidate.
