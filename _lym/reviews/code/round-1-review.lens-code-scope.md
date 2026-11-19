```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each change in the diff to the goal points of the board entry.

**Decisions** (decision record alone; no `Added after Discussion` entries):
- G1 — Expected: `pr-findings` builds the recipe's shed with its fabric dependency set (`OpenFabric`), so the nil-seam refusal is gone. Built by C1, C2 and C3.
- G2 — Expected: `pr-findings` runs the findings round on a valid findings file. Built by C1, which removes the error that stopped the round. The rest of the verb is unchanged.

**Changes**
- C1: `internal/loomcli/prfindings.go` +285–306 and −320–333. The edge closure becomes the method `prReviewSendBackEdge`, which calls `c.wireLanding()` before `c.newShed()`. This traces to G1 and G2. Moving the closure into a method is a refactor forced by the fix, which needs a method a test can reach. It is not unrequested scope.
- C2: `internal/loomcli/doc.go:42` and `:608`. Both lines name the new caller of `wireLanding` and say the edge builds a shed with the landing seams wired. C1 forces both, because it makes the old line 42, which listed only `run` and `step`, stale.
- C3: `internal/loomcli/prfindings_integration_test.go`, new file. It is the regression test for C1, which C1 forces.
- C4: commit `22b3bee0`, which re-wraps C1's comment and seeds stencils in C3's test. This is a fix-up of C1 and C3 and adds no new purpose.

Every change traces to G1 or G2 or is forced by one that does, and each goal point is built by at least one change. No commit in the range carries a `Pattern-Departure:` trailer, and none is needed.

**Findings:** none.

**Tool calls:** none. My context already held the diff, the decision record and the commit log.
