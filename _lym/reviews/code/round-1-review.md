---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: handler
    location: internal/battencli/lifecycle_integration_test.go:1417
    summary: startCostBound still compares a measured real time to a fixed bound, which the PATTERN bullet this diff adds forbids with no exception
    fix: In pattern/PATTERN-test-speed.md under "Time is injectable", add to the new bullet that a cost budget kept by a `//testtiming:keep` marker is the one exception, or rewrite stepStartCost_CommandRowsCostUnderTheBoundBeyondTheCommitAndTheProcess to assert an outcome instead of a time. Either way the rule and the test must agree.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/battencli/lifecycle_integration_test.go:313
    summary: The testtiming:keep marker still says a Command row costs under 100 ms, but startCostBound is now 2 s
    fix: Reword the marker on TestBattenIntegration_Rows to the new bound, or to its purpose (a Command row that waits seconds on a remote or a fixed delay). Keep it consistent with the startCostBound comment.
---

### [MEDIUM] startCostBound is still a measured-time comparison

**Location:** `internal/battencli/lifecycle_integration_test.go:1417`

**Issue:** The goal asks to make each elapsed-time test load-proof.
`startCostBound` rose from 100 ms to 2 s, but the step still compares a measured wall time to that bound.
The bullet this diff adds to `PATTERN-test-speed.md` says no test does so.
The summary admits the bound "cannot be fully load-proof".
A loaded machine like the one in the goal's report, where a 5 s launch took 6.18 s, can still reach 2 s.
The raise also weakens the test: a cost regression of tens of milliseconds, which the 100 ms bound caught, now passes.
The rule and the test disagree, so one must change.

### [LOW] Stale testtiming:keep marker

**Location:** `internal/battencli/lifecycle_integration_test.go:313`

**Issue:** The marker's justification cites a bound "under 100 ms" and the decision record's D2 criterion.
The constant it describes is now 2 s, so the marker misstates what the step pins.

## Rejected

- Code-tests lens item 1 (`TestResolveToken_Chain` lost its timing check): the seam is faked, so nothing can wait, and the lens itself raised no defect.
- Code-tests lens item 2 (`startCostBound` weakens the pin): folded into F1.
- Code-tests lens item 3 (late-runner test fails only after 2 m on a regression): the exit code tells the paths apart, so the regression is caught; a slow failure is the accepted trade the summary names.
- The `loomcli` driver-bootstrap test is not in the diff: commit 2ad39d26 is an ancestor of HEAD, and the test already holds the stub on a release file with a 60 s `launchDeadline`. This is no scope gap.
- The remaining elapsed-time comparisons in `shuttleengine`, `loomshed`, `sheddriver`, `tenderengine`, `reportengine`, `runproc`, `hibernateengine` and `creelengine` run on fake clocks, so load does not move them. The goal does not ask to change them.
- The card-call race in `gatecli` the summary names compares no measured time and is outside the entry.
- No `Pattern-Departure` trailer is needed: the commits depart from nothing the entry asks for.
- The Windows-only `treadleengine` test was compiled, not run, as the summary states; the verify gate covers the rest.
