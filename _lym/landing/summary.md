# Carry Drift's regression verdict into the PR and landing commit

A clean Drift-Verify compare left the pull request asking the operator to run by hand the compare Drift had already run.
The description was written before Drift-Verify ran, and neither it nor the PR-Review agent's prompt knew the outcome.

Now `internal/driftengine` renders a `## Regression check` section from the task's Drift scope record, and the landing carries it into every place a reader looks.

- The section states the scope (`full`, `partial` with its skipped tests, or `off`), the commit the baseline was captured at, the last compare's result (passed, showed drift, not run, or waived with its reason), and each accepted drifted test with who accepted it and why.
  An acceptance the run's parent has not yet ruled on is marked as such.
- `PRReview` appends the section after the description body when it opens or refreshes the pull request, and `Finalize` appends it to the landing commit message, in both cases before the credit block.
- The PR-Review session's prompt and the Describe prompt both show the section.
  Describe is told the check has already run, to write no regression section of its own, and to drop or rewrite any hand-check item the check settled.
- `Closes #N`: none; the task resolves no tracked issue.

## Design decisions

- The section is appended in Go, not written by Describe.
  Code renders it exactly from the record, where an agent's prose can drift from it.
  The summary artifact never holds the section, so it reaches the PR and the commit whichever producer wrote the artifact last.
  `contracts/specs/loom-final-summary-spec.md` records this.
- The verdict also goes into the landing commit message, beyond the PR, so it stays in main's history once the PR is gone.
- Drift-Verify now records the compare in `ScopeRecord` (`BaselineHead`, `ComparedHead`, `ComparedAt`, `ComparePassed`), on both the deterministic pass and the agent's `Done` outcome.
  `Verdict` returns `""` without touching the ephemeral drift directory when no scope record exists.
- `landingshed.Deps.DriftVerdict` is a nil-is-absent seam, wired in `internal/loomcli` to read the record at call time, so it sees a compare recorded after bootstrap.
  `internal/testkit/envkit` and both wiring seam tests list it as a legal nil seam.

## To check by hand

- `Landing.DriftVerdict` is exempt from the nil-seam guard in `internal/testkit/envkit/envkit.go`, `internal/battencli/wire_seams_test.go` and `internal/loomcli/wiring_seams_test.go`; confirm the exemptions are intended and `internal/loomcli` still fills the seam.
- `Finalize`'s merge-commit message with the section (`internal/landingshed/finalize.go`) is covered by tests added late; confirm they pin the section's position before the credit block.
- The section is appended only where a producer reads it, so a summary left from a run that skipped Describe can still carry a stale hand-check line.
  The appended verdict sits next to it and contradicts it; confirm that is acceptable (`internal/landingshed/describe.go`, the Describe stencil).
- Not run: an end-to-end landing on a real hub with a clean compare, to see the section in the opened PR and the PR-Review prompt.
