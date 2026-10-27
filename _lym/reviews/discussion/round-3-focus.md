---
round: 3
focus:
  - "Refusals of placeDobby (internal/beamcli/cli.go) after lymcwd.ResolveWorktree has succeeded, such as RequireDrivableWorktree, PrimeName, ReadOrigin and DobbyGeometry: does the decision record say where a re-exec'd image's refusal is logged (the worktree's .lym/beam dobby.log, by the path rule in internal/hubgeom/dobbygeom.go) and posted through orchcli.NotifyFromScoped, with an Acceptance criterion that asserts it, and what bounds the case where no worktree resolved?"
  - "The re-exec test of the decision record (this pid and this process's start ticks): does the same test govern recordStart and tick's refill of a record found without a pid in internal/dobbyengine/loop.go, so a fresh dobby that reuses a dead dobby's pid is not recorded as a re-exec and every writer of State.PID writes the start ticks beside it, and does an Acceptance criterion cover a ticks-less record?"
  - "A dobby started by a binary from before this change holds dobby.log as its stdout and stderr and keeps them across re-exec (SpawnDetached, syscall.Exec): does the record say where such a dobby's raw output goes, and is the rotation of dobby.out at a spawn consistent with its rule that only the lock holder rotates?"
  - "The worktree-gone exit of a beam dobby, whose dobby.log lies inside the removed beam worktree: does the record name where the exit line lands without recreating the removed tree, and what it does with a tidy agent in flight in State.Tidy?"
  - "Errors that Run in internal/dobbyengine/loop.go returns before its loop (lock directory, lock acquisition, recordStart) after a re-exec onto a new binary: does the record post one notice for them as it does for a wiring failure, and does a refused re-exec that finds another pid in the record when its wait ends avoid a false alarm?"
  - "The adopt check process spawned by lym adopt (internal/adoptcli/deps.go, startSuite): does the record state where the child's own stdout and stderr go so a losing check does not append to the running check's suite.log and a panic is on disk, and does the spawn-failure way forward name a step that migrates adopt before lym gate test --gate adopt reads it?"
---

The focus lists where the round 2 findings clustered, stated without reference to the review.
Check each entry against the rubric's Do-not-flag list before raising a finding.
