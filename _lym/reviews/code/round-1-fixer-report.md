# Fixer-report — code review round 1

## Changes

Commits `87f1919c` (F4), `cdc17965` (F1, F2, F3) and `1c5b3b5e` (self-check findings).
F1, F2 and F3 share `tierrun.go` and `tierrun_test.go`, so they sit in one commit rather than three.
`lym gate test ./internal/gatecli ./internal/fabricengine` passed before the self-check.
The two self-check edits (a stencil sentence and a file header) were checked with `go vet ./internal/fabricengine` only.

- **F1, fixed as proposed.**
  The two stencils (`darn-fragment-make-change.md`, `webster-body-implementer.md`) and the way-forward clause of `noTestNotice` offer the hand run only while the hub's `refuse_raw` does not list `dotnet test`, and otherwise tell the agent to report the skipped tests to its parent.
  The stencils' "never a raw test command" rule stays, with the hand run as its one stated exception.
  `TestRunTierNoTestNotice`'s expected line matches the new text.
- **F2, fixed differently.**
  Not made: detecting a disabled project inside a multi-project invocation.
  Reason: `vstestCounter` keeps only summed totals, and its summary lines name an assembly (`x.dll (net8.0)`), not a target.
  A `.sln` target selects projects the gate does not parse, and a multi-targeted project prints one summary line per framework, so a summary-count comparison would miss real skips and misreport others (`counters.go:28`, `counters.go:456-470`).
  Made instead, the review's own fallback: the line now says the gate reads only the invocation's total, so it names no property and sees no skip inside an invocation that ran some test.
  `doc.go` states what the line covers (an invocation whose whole output reported no test) and what it cannot see (a disabled project beside one that ran, a gated class in a project that ran other tests, and the property behind a skip).
  The `noTestNotice` comment carries the same limit.
- **F3, fixed as proposed.**
  `TestRunTierNoTestNotice` gained a row for an invocation with no targets, whose line names the run directory, and a row for a failing invocation, which prints no line.
  The command now quotes `{targets}` and has a `*bad*` arm that exits 3, and a `wantExit` field carries each row's exit code.
- **F4, fixed as proposed, in the second place the fix key allows.**
  A new `internal/fabricengine/codelocation_test.go` pins both unchanged-location paths of `CodeLocation`: a worktree that is no weft sibling, and a weft sibling whose code directory is absent.
  It is not a row of `TestGateTestMap`, because that hub is shared and removing the pair's code directory would break the rows after it.

- **Gate repair, after the report.**
  The gate flagged the comment lines this round added to `doc.go` and `tierrun.go` for a fixed-column wrap and for lines of 120 characters or more.
  Each is now broken at sentence boundaries, one sentence per line, with no change of meaning.
  The gate had run no test; it was not re-run for this comment-only change.

## Self-check

- fix-consistency, LOW: `codelocation_test.go` lacked a file header. Fixed as proposed in `1c5b3b5e`.
- fix-proposal, LOW: the darn stencil's "report to your parent" gives a parentless darn writer nothing to do. Fixed as proposed in `1c5b3b5e`: it now says to name the skipped tests in the change description when there is no parent.
- fix-code-correctness: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
