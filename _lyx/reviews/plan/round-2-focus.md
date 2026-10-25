---
round: 2
focus:
  - "Every weft pull or fast-forward path in internal/fabricengine, including Fabric.fastForwardWeft in parenttips.go reached from FastForwardPair in internal/landingshed/finalize.go: does some card guard it with the marker comparison and the standing-move-record stop, or does it call PullWeft unguarded?"
  - "Cards that prescribe a refusal or failure path after the anchor move commit (Apply errors, dobby restart, ensureOne gate): does each name an end state and a landed re-run, and can the prescribed restart pass the gate that refuses while a move record stands?"
  - "The drop-local reset: does the plan's comparison refuse whenever origin's tip carries a marker other than the checked-out one, including when the local weft is ahead of origin with an unpushed re-anchor?"
  - "The create-at-anchor path in Topology.add: does the plan name one engine path for the move, and is the order of record removal against createSteps' push, workspace and dobby steps stated?"
  - "Any deletion or working-tree restore in internal/fabricengine the plan prescribes: does it go through a destroy.go executor per PATTERN-fabric-git?"
  - "Refusals in the plan for owner, unfinished run and invalid anchor targets: does each name a next step that progresses rather than loops back to another refusal?"
---
