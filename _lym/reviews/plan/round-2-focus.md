---
round: 2
focus:
  - "Card 10's tests: check that every tier0 test row spawns no process; taskWorktreeGone resolves a path through lymcwd.ResolveWorktree, which runs git, so any row past the absent-path row belongs in a tier1 hubforge test."
  - "Card 4 (adopt-only Add): check that every closed-set refusal the plan promises is reachable for a task branch that exists only on origin, with the local weft branch checked out in another records worktree; refuseAdoptLocalCheckedOut in internal/fabricengine/add.go runs only under opts.AdoptLocal && warpAdopted."
  - "Card 4's reused records worktree: check that advancing a local weft branch behind origin (fastForwardTo from probeLiveLocalWeft) cannot be a fetch into refs/heads/<weft> while a worktree has that branch checked out, which git refuses."
  - "Card 6's archiveWeftTip steps: check that each ancestry comparison runs only between commits both present in the records repository, in the order the card states, including when another clone moved origin's weft tip."
  - "Card 8's two sweeps (Topology.Cleanup, cleanupRemoteWarp): check that the state the per-slug try lock guards is read or re-read after the lock is held, so an Add that finishes between enumeration and lock cannot lead to deleting a task branch a live pair just took over."
  - "Card 1's retry predicate: check that adding the 'missing necessary objects' reason does not change the classification or recovery path of PushRebaseFree and Push, as the plan licenses that retry for the archive-tag push only."
  - "Card 5's PushPairAnchored no-op under an exclusive teardown hold: check that each caller reading a nil return as pushed (the landing PushTask in internal/loomcli/landingdeps.go, commit-records, lym fabric push) has a stated bound on what the skip lets through."
  - "Card 13 and batten's refusal texts: check that every refusal outside fabric which quotes a fabric way forward naming lym fabric remove (battencli's stoppedRefusal path from internal/battencli/resumereconcile.go) also names the cancel or Land way forward the plan requires, and that the removal warning has a typed path into the fabriccli envelope."
  - "Cards 1, 2, 6, 8, 10 and 13: check that doc comments, file headers and background text the card's change falsifies are in the card's Edit list and Intent, including the gone-pair refusal text in internal/battencli/wire.go that says batten never re-creates a pair."
  - "Card 3's config.lock test row and card 5's PushAnchored rows: check that each named input reaches the failure it asserts, and that each tested path has a production caller."
---

The focus lists where the plan's guards, orderings and tier placement are most likely to break; it is no cap on what the reviewer reports.
