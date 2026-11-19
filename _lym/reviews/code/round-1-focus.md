---
round: 1
focus:
  - "internal/loomcli/prfindings.go prReviewSendBackEdge: does calling c.wireLanding() before c.newShed() fill every seam the landing rows' constructors require (OpenFabric and any other Deps field), and is it safe when the edge runs more than once or after wire already filled them?"
  - "internal/loomcli/prfindings.go: does the edge's c.wireLanding() leave the PR-Review row's session seams as the verb needs them, or does it alter state (watches, registrations, process records) the old edge never touched?"
  - "Other callers of c.newShed() in loomcli (status, pause, other verbs): does any of them build a shed with the landing rows and hit the same nil-seam refusal the issue describes?"
  - "internal/loomcli/prfindings_integration_test.go: does the test fail on the pre-fix code, and does its want of true, empty reason hold for the tapestry recipe's real budgets? The reach report lists the new function as reached by no mapped test."
  - "internal/loomcli/doc.go: do the two edited doc lines match what the code now does, including the claim that wireLanding is passed by the send-back edge?"
---

The issue is a single nil-seam failure in the `pr-findings` send-back edge.
The diff moves the edge into a method, wires the landing seams first, and adds one integration test.
