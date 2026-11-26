---
round: 2
focus:
  - "Worktree-Teardown with FABRIC_SKIP_GIT or FABRIC_SKIP_PUSH set (the Teardown.Push and Teardown.Level closures in internal/battencli/wire.go, the loop in internal/battenshed/teardown.go#Call): does the row always end, removing the pair or going Stuck, with no unbounded re-push or prime-lock re-take, and can any path remove a pair whose commits are not on origin while pushes are on?"
  - "The same loop with pushes on: when the level read stays false after a push that returned nil, or turns false again because the run committed in between, does the bound on re-pushing give the right outcome and a refusal that names its way forward?"
  - "The teardown's push of a present pair: do tests drive the real wiring, including the mapping of an ended entry and of a hand-over to the producer's sentinel errors, so that a broken mapping fails one?"
  - "Pusher starts from the dobby tick (internal/battencli/dobbyreads.go#DobbyRuns, the hub Tick in internal/beamcli) and from resume (internal/battencli/wire.go, internal/loomcli/wiring.go): would a test fail if a start call were deleted or passed the wrong target, and do any seams added for this leave production wiring unchanged?"
  - "internal/runpusher: are the real lock, the NewDeps stop conditions, the worktree-gone, pause and hand-over reads and the restart's failure logging each pinned by a test that would fail on a broken implementation?"
---

The entries cover the teardown's termination and safety, and whether tests reach the real pusher wiring.
