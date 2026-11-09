---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/battencli/lifecycle_integration_test.go:1417
    summary: startCostBound of 2s is not far above what load adds, so the step does not obey the cost-budget exception as written
    fix: In internal/battencli/lifecycle_integration_test.go raise startCostBound to 30 * time.Second, and reword its comment to say it catches a row that waits tens of seconds on a remote or a fixed delay while the row's own work takes milliseconds. Keep the sentence that the bound sits far above what load adds, since it is then true. The testtiming:keep marker on TestBattenIntegration_Rows needs no change.
---

### [MEDIUM] startCostBound is within reach of a loaded machine

**Location:** internal/battencli/lifecycle_integration_test.go:1417

**Issue:**
The rule in `pattern/PATTERN-test-speed.md` under "Time is injectable" forbids a test comparing a measured real time to a bound a loaded machine can reach.
Its one exception allows a cost budget no outcome can stand in for, when "the bound sits far above what load adds to the work it measures".
The step `stepStartCost_CommandRowsCostUnderTheBoundBeyondTheCommitAndTheProcess` qualifies for the exception on the first condition:
a row's cost beyond its commit and process is a pure timing quantity, and no exit code, error or pid separates a slow row from a fast one.
It fails the second condition.
The decision record's own incident was a launch measured at 6.18 s on a loaded machine, so 2 s is not far above what load adds.
The measured cost is also noisy by construction.
`processTime` is taken from one earlier run of the command and subtracted from a later, separately timed step.
Load that makes the later process slower than the earlier one lands in `cost` and counts against the 2 s.
The new comment on `startCostBound` states that the bound sits far above load; the evidence in the decision record does not bear that out.

The marker question the focus names is settled: the `testtiming:keep` marker on `TestBattenIntegration_Rows` and the `startCostBound` comment agree.
Both name the bound on a Command row beyond its status commit and its process, and both treat it as a wall-clock bound no other test asserts.
The marker carries no number, so it cannot go stale against the constant.
The marker states the bound's subject but not its purpose, which the constant's comment states ("catches a row that waits seconds"); that is not a disagreement.

After F1 is fixed the comment's "seconds" wording must follow the new bound, or the two disagree.

## Rejected

- The `runRowCommand` timeout step now sleeping 300 s against a one-minute assertion obeys the rule: the sleep outlasts the deadline by a release-free margin far past it.
- The new exception bullet's own wording is the answer key's rule text, not the subject of a finding.
