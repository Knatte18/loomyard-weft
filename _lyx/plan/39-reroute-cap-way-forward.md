# Card 39 — reroute-cap-way-forward

**Create:**
- `plan:internal/loomcli#TestLoomRerouteCapWayForward` -> `func TestLoomRerouteCapWayForward(t *testing.T)`

**Edit:**
- `internal/loomcli#loomRerouteCapWayForward`

**Intent:** The reroute-cap halt's way-forward clause names Finalize's causes that are not a parent settling (D11).
`loomRerouteCapWayForward`, the told `Shed.RerouteCapWayForward`, keeps its parent-branch and Webster plan-conflict clauses and gains two: for a fixer whose attempts are spent, fix the failing tests on the task branch, then `lyx loom resume`; for a fixer whose change is over the caps, review the fixer's commits and fix on the task branch, then `lyx loom resume`, as the other over-cap halts say.
Both ways forward are ones no agent's settings deny (`PATTERN-no-denied-recovery`).

`TestLoomRerouteCapWayForward` builds a loom shed over a Finalize producer that reroutes four times with no Done between and asserts the run halts blocked at Finalize with a reason starting `reroute cap reached` whose way forward names the parent-branch cause, the spent fixer cap with its way forward, and the over-cap fixer change with its way forward.

**ImpactSummary:** One told clause grows; it is read by every loom and darn reroute-cap halt, Webster's included, whose own clause is unchanged.
