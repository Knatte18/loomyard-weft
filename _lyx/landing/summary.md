# An adopt starts the finalize gate on the adopting machine

The operator's machines differ (Ubuntu 26.04, Ubuntu 24.04, Windows 11), so code landed on one can break on another.
Until now nothing checked the landed code on the adopting machine, and a break showed only inside the next run's gate.

A `lyx adopt` call that records this machine as the owner of an entry or of the prime now ends by starting `lyx gate test --gate finalize --wait` in the prime, as a detached process.
The adopt returns at once.
The suite's output goes to `_board/.lyx/adopt/suite.log`, truncated at each start, and the envelope's new `suite` field carries the pid and the log path.
A spawn that fails sets `suite_warning`, naming `lyx gate test --gate finalize` in the prime as the way forward, and changes no slug's outcome.

## Design decisions

- The full suite is the hub's `finalize` gate key, so it follows the hub's own whole-tree landing check, and the manual LLM tier stays out.
  A tier the green record already holds green for the prime's tree is carried and runs nothing.
- An adoption session is one `lyx adopt` call: the call starts at most one suite, after the dobbies, however many entries it adopts.
  A call that records no owner, or that is refused, starts none.
  Two separate adopt calls each start a suite, and the gate slots serialize them.
- A red suite only reports, in the log and the gate's own run records.
  It cannot refuse the adopt, since the owner write and the resumes have already taken effect when it ends.
- The suite runs detached because the full suite takes many minutes.

## Changes

- `internal/adoptcli`: a `StartSuite` seam on `adoptDeps`, called last in `adopt`, and `startSuite`, its hub implementation, which logs the spawn and starts nothing under `go test`.
  `cli.go` adds the `suite` and `suite_warning` envelope fields and a step 9 to the long help, which is trimmed to stay under the help-length cap.
  The package doc describes the new step.
- `internal/lyxdirs`: `AdoptDirName`, the directory of the suite's log under the hub's board-level `.lyx`.
- `docs/overview.md`: the adopt module row names the suite start.

## Tests

`TestAdopt` gains rows for one suite start after the resumes and the dobbies when the call adopts the prime and two tasks, for no suite start when no owner is recorded, and for the failed-start warning beside an adopted slug.
`TestStartSuiteStartsNothingUnderTest` pins the guard that keeps a test binary from re-executing itself.
