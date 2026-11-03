# Add Drift-Recheck: compare again code committed after Drift-Verify

Code that a row after Drift-Verify commits, such as a Code-Review fix, used to land with no compare.
`lym drift compare` refuses outside Drift-Verify and skips accepted tests, so nothing could check the new code or the accepted cases' new values.
On the `cs` hub a Code-Review fix rewrote `ResultFormatter` after Drift-Verify had accepted five drifted cases, and the accepted values then described code the PR no longer held.

A new mechanical row, `Drift-Recheck`, now follows the Code-Review perch in the darn and tapestry recipes, before Describe.
It spawns no agent and carries the `drift.enabled` entry gate, so a hub with regression off records it Skipped.

## Behavior

- `Drift-Recheck` is Done with no compare when no scope is recorded, the scope is off, the scope is waived, or HEAD is still the commit of the last compare.
- With no test accepted, it compares HEAD against the baseline.
- With tests accepted, it compares HEAD against an accepted profile, so an accepted test whose output differs from what its acceptance saw counts as drift again.
- A passing compare is recorded in the scope record and the row is Done.
- Stuck cases: no compare recorded yet, a dirty tracked tree, an undeclared compare step, and drift.
  The recipes route the Stuck to `Drift-Verify`, which judges the drift again, and the perch then reviews that row's fixes in turn, under the shed's bounce budget.
- Drift against the accepted profile, or no profile captured at the last compare's commit, lapses every acceptance first, since the compare cannot tell which accepted test changed.
  Under scope full the parent must rule on each lapsed acceptance again once Drift-Verify's agent accepts it anew.

## Design

- The accepted profile is a capture of the current code taken into a new `.lym/drift/accepted` directory (`Paths.AcceptedDir`).
  `Drift-Verify` takes it when it ends Done with tests accepted, through `RunAcceptedCapture`; a capture that does not pass halts with `lym loom resume` as the way forward.
- `ScopeRecord.AcceptedProfileHead` names the commit the profile was captured at.
  `Accept`, `Withdraw` and a lapse clear it, so a profile never outlives the acceptances it holds.
- `RunCompare` takes an optional `ProfileDir`, which `{baseline_dir}` renders to; empty keeps the baseline directory.
  Drift-Recheck's accepted compare leaves out only the scope's skipped tests, so accepted tests are compared too.
- `RunSideCapture` and `RunAcceptedCapture` share `captureInto`.
- New seams `Deps.CaptureAccepted` and `Deps.CompareAccepted`, wired in `internal/loomcli/driftwiring.go`; `shedrecipe` registers the `DriftRecheck` engine and `loomshed.NameDriftRecheck` is its row name, with an interrupt policy of reinvoke and a report phase beside Drift-Verify.
- `lym drift compare` still refuses outside Drift-Verify; the recheck is a row, not a new verb.

## In-flight runs

A tapestry run parked inside the Code-Review perch, whose recipe exited to Describe, now exits to `Drift-Recheck`.
With no compare recorded it sends the run back to `Drift-Verify`.

## To check by hand

- The wiring of the accepted-profile seams, `CaptureAccepted` and `CompareAccepted` in `internal/loomcli/driftwiring.go`, was flagged as untested and the finding is still open: confirm the integration tests in `internal/loomcli/driftwiring_integration_test.go` exercise them against a real capture and compare.
- No end-to-end run on a hub with a real capture and compare command was done: run a darn task with `drift.enabled` and an accepted test, change the code in a Code-Review fix, and confirm `Drift-Recheck` compares against the accepted profile.
