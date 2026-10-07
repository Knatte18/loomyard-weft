# Card 9 — each verify site reports a timed-out verify as a failed gate

**Edit:**
- `internal/loomshed/verifygate.go`
- `internal/websterengine/verifygate.go`
- `internal/websterengine/verifygatereport.go`
- `internal/websterengine/verifygate_test.go`
- `internal/websterengine/verifygatereport_test.go`
- `internal/landingshed/verifygate.go`
- `internal/landingshed/verifygate_test.go`

**Create:**
- `internal/loomshed/verifygate_test.go`

**Intent:**
An expired verify timeout rides each site's normal failed-gate path, with findings naming the timeout, the log path and the log tail (D3).
It is never a returned error, so no site turns it into an infrastructure fault, and a caller's own cancel stays a returned error as today.

- `loomshed`'s Webster-Burler round gate: `verifyFailureFindings` renders a `TimedOut` result as "The verify command did not finish within <timeout> and was killed." in place of the exit-code sentence and the shell-could-not-start clause, then the full log path and the log tail as for any failure.
  The pure renderer is pinned by the new untagged `verifygate_test.go`, since the package's only test file is integration-tagged and cannot shorten the production timeout.
- webster's gate (`newVerifyGate`): a timed-out first run is not rerun (a hang is not flakiness, and a rerun would double the wait); it fails at once with a report carrying the new `VerifyGateReport.TimedOut` (the timeout as a duration string), the log path and a bounded log tail (`LogTail`), and `renderVerifyGateFindings` leads with the timeout sentence when it is set.
  A first run that failed normally and a rerun that then times out fails the same way.
  `verifygate_test.go` adds the rows over the seam fakes (first-run timeout fails without a rerun, rerun timeout fails), and `verifygatereport_test.go` the rendering and the round trip of the two new fields.
- `landingshed`'s Publish and Finalize merge-in verify (`verifyGate.check`): a `TimedOut` result is the Stuck reason `verify did not finish within <timeout> after merging parent branch "<branch>"; output: <log>; fix the hanging test on the task branch, then resume`, kept on one line like every Stuck reason, so the log path is where the tail is read.
  It is tested before the existing `ExitCode < 0` branch, whose could-not-start wording is otherwise wrong for a timeout; `verifygate_test.go` adds the row.

`webstercli`'s `verify` verb needs no change beyond card 8: its findings already carry `exit_code` and `detail`, and `detail` names the timeout.
A grep for each site's existing failure wording finds no test asserting it that this card's new branch changes, since every existing wording is kept for the non-timeout cases.

**ImpactSummary:** Three gate renderers gain a timeout branch; every existing failure message is unchanged.
