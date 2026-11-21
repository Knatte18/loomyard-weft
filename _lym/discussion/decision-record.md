## Goal

```markdown
## Verify triage names the verify step as a flaky test and drops its output

Imported from [issue #160](https://github.com/Knatte18/loomyard/issues/160).

The integration-suite triage after a `lym webster` verify run wrote a friction note listing `verify-command` as a non-regression failure that "failed the first verify run and passed or vanished on rerun (flaky)".
The only other line in the note is "Shell cwd was reset to <worktree root>", which is the harness's cwd-reset notice, not a test result.

This is abnormal because the identity `verify-command` names the verify step itself rather than a test.
A failure of the step itself, possibly caused by the cwd reset, is classified as a flaky test, so a lym-side verify problem is hidden as test flakiness.
The note does not say what the first run's failure was, so the cause can't be recovered from it.

What to do:
- Check how triage derives identities and whether `verify-command` can stand for a failure of the command rather than of a test.
- Keep the first run's failure output in the triage note.
- Keep a cwd-reset notice out of the failure identity and the note body.

Provenance: task `miqp-baseline-oracle`, note `webster-verify-triage.md`.

## Flaky internal/reedcli failure on the first verify run

Imported from [issue #158](https://github.com/Knatte18/loomyard/issues/158).

The webster verify triage reported one non-regression failure that passed or vanished on rerun: the package `github.com/Knatte18/loomyard/internal/reedcli`.

The package failed the first verify run and was then classed as flaky because the rerun did not reproduce it.
That is abnormal: a verify run should be deterministic, and a flaky package makes verify fail spuriously and costs a rerun.
The note records no failure output, so the cause is unknown.

What to do: run the `internal/reedcli` tests repeatedly, for example with `lym gate test ./internal/reedcli`, to reproduce the failure and find the nondeterminism, such as shared state, ordering, timing or a leaked environment.
Consider having the triage note keep the first run's failure output so a flaky identity can be diagnosed.

Provenance: task single-repo-fabric-mode, note webster-verify-triage.md.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-21): No test seam for the red-first overlay: the diff changes a non-Go spec

Decision: Waive the red-first gate for this run and keep the spec commit 87f6be00.

Rationale: The diff updates contracts/specs/webster-spec.md, which the red-first overlay cannot restore on the merge base. The Go test TestVerifyGate_FlakyFailurePassesOnRerunWithNote reaches the bug through base code (newVerifyGate and VerifyGateNotes.Apply). A stale spec is worse than a waived gate; the overlay gap is filed as note red-first-overlay-non-go-files.

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - command-failure-only-note-header-untested (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
