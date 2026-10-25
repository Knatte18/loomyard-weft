---
round: 2
exclude_lenses:
  - board-fit
focus:
  - "D7's step order in decision-record.md: runproc.End (internal/runproc/end.go) ends every ppid descendant of a record, spawnPass in internal/beamcli/dobbydeps.go starts passes as dobby's children, and fabricengine registers records pushes under runproc.HubOwner. Settle whether the steps and refusals, for a worktree and for the prime, still leave in-flight passes and pushes running as the acceptance criteria claim."
  - "D7 and D10 for the prime on a hub with primes on several machines: the prime is checked out on every machine and shares main-weft, and internal/fabricengine/remove.go refusePrimeSlug refuses removing the prime. Settle how another machine's prime follows a committed marker change, and whether every refusal's way forward names a step that exists."
  - "D7's recovery after a kill at each point (before the marker commit, between the commit and the wiring, after the wiring): settle whether the source anchor and the settled state are determined from durable evidence, including a child worktree forked after its parent's change, and what internal/fabricengine/reconcile.go repairPairWiring does to such a pair."
  - "D7's lock and refusal bounds: callers of dobbyengine.EnsureLive in internal/orchcli, internal/adoptcli and internal/beamcli, and child runs landing into the moving worktree under internal/parentlock. Settle what each can do while the move runs and what bounds it."
  - "D8's census and Bolt marker removal, and D10's pull refusal: settle what a branch that is unfetched, lagging, empty-marked or unpushed does to the census, and which tip the pull refusal compares when the local weft is ahead of origin."
---

Round 1 findings were all design-class at MEDIUM or worse in the D7, D8 and D10 areas, so those areas stay in focus.
The board-fit lens returned nothing in round 1 and is settled.
