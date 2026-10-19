---
round: 2
focus:
  - "Card order: in `_lyx/plan/01-hub-workspace-file.md`, check that every assertion the card plans holds with the repo as it stands after that card alone, given that `Topology.add` in `internal/fabricengine/add.go` still creates `_portals` links until the card that retires portal creation. Settle whether any test step through `hubforge.AddPair` in `internal/ideengine` makes an earlier card's assertion fail."
  - "Hub containment: in the hub workspace folder listing the plan prescribes, settle whether a linked worktree registered outside the hub directory can yield a folder path leaving the hub, given `PATTERN-hub-containment` as declared changed (the file references hub-inward folders only), and whether the card's tests reach the skip."
  - "Destruction chokepoint: in the plan card that sweeps retired hub artifacts and empties `_portals` directories, settle whether every removal passes the ownership and dirtiness checks `PATTERN-fabric-destruction-chokepoint` requires before `removeContainedPath` runs, whether an ownership kind admits the `_portals` container, and whether a refusal or I/O error from any removal is surfaced rather than discarded."
  - "Doc and test-header coverage: settle whether every doc comment or file header naming the workspace file under `_launchers/`, or naming the removed portal accessors, is targeted by some card, and whether `## verify:` runs each package such a card edits."
  - "Reachability: for each refusal or failure row the cards prescribe, settle whether the named test input reaches that branch, including on Windows, and whether any concurrency row would fail without the lock it claims to test."
---

Round 1 left three BLOCKING design findings; round 2 reviews the plan as rewritten, with fresh eyes on the questions above.
