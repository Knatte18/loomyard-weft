# Relevant code

- `internal/gatecli/green.go` `openGreen`, `headIdentity` — the green record keys on HEAD's tree and commit and refuses a dirty worktree
- `internal/verifytree/verify.go` `Verify`, `headTree`, `headCommit` — verify records key on HEAD's tree and commit
- `internal/fabricengine/merge.go` — merge verbs, whose package comment says the weft is not a merge participant
- `internal/fabricengine/parenttips.go` `Fabric.HoldsWork` — tree-based held-work check D9 strips
- `internal/fabricengine/replay.go` `ParentHoldsTaskWork` — held-work reader and the replay D7 limits
- `internal/fabricengine/originreads.go` `OriginParentHoldsTaskWork` — catch-up's held-work read
- `pattern/PATTERN-fabric-boundary.md` — excludes, weft commits, teardown and the Bolt `.gitignore` the layout touches
- `pattern/PATTERN-lym-files.md` — per-branch records rule and the `_lym`/`.lym` split
- `pattern/PATTERN-cwd-resolution.md` — anchor marker resolution through the weft sibling
- `pattern/PATTERN-pair-remote.md` — push order, teardown archive tags and the prime's records-only push
- `pattern/PATTERN-run-fsm.md` — transition commits, owed pushes and hand-over
- `pattern/PATTERN-owner-machine.md` — origin reconcile and catch-up both layouts must keep
- `pattern/PATTERN-gitrepo-client-boundary.md` — no force-add and the remote-refs lock for the view push
- `pattern/PATTERN-landing.md` — merges only in Finalize, one pull-request resolver
