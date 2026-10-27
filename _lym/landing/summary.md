# Keep test trace logs out of the live hub and out of Bolt

A gate run's `fabriccli` test binary wrote `trace-*.log` files into the live hub's `_board/.lym/logs`, and the next `board sync` committed them to Bolt.
This change closes both halves: the test no longer resolves the real hub, and Bolt's stage-all commit never takes a file under `.lym/`.

## The leaking test

`TestRunCLI_HubMutationScenario`'s bypass-push step called `fabriccli.RunCLI`, which reads the process cwd.
Under `go test` that cwd is the package directory inside the real hub's task worktree.
The bypass branch resolves the hub from cwd and calls `logger.SetDurableSinkDir(fabricengine.HubLogsDir(hub))`, so the test binary's process-wide trace sink moved to the live hub, and every later record of that binary followed it.
The leaked files carry the scenario's own temp paths and an empty `worktree_root`, the shape that redirect leaves.

The step now runs through `fabriccli.RunCLIIn(h.BoardDir(), …)`, the cwd the real detached push child starts in, and resets the sink on cleanup.
It also asserts that `logger.TraceDir()` is the test hub's `HubLogsDir`, so a regression fails the scenario instead of writing into whatever hub holds the checkout.

## Bolt's stage-all commit

`commitWeftAt`, the wildcard-stage commit behind `Bolt.Commit` and so behind `boardengine.Sync`, now calls `seedWeftArtifactExcludes` before staging.
That function owns the weft's `.git/info/exclude` lines: `.lym/`, `.weft/`, the push lock and the `*.lock`/`*.swaplock` patterns.
Clone, reconcile and the weft verbs already seeded it, but a board whose common gitdir lacked the lines had nothing to repair them before a sync, and the stage-all took whatever lay under `_board/.lym/`.
A seed failure now fails the commit, since committing without the excludes is the defect.

`TestBolt_DirtyRepo_CommitsAndPushes` plants `.lym/logs/trace.log` beside the board file in an unseeded repo and asserts that `git ls-files` lists only the board file after the commit.
`TestBolt_CleanRepo_CommitAndPushAreNoOps` no longer removes `.gitrepo-push.lock` by hand, because the seeded exclude now covers it.
The stale clone comment that said Bolt's commit seeds nothing is rewritten, and fabricengine's `doc.go` states the new seeding.

## Design decisions

- The exclude is seeded in fabricengine rather than adding `.lym/` to the `_board/.gitignore` patterns `boardengine` writes, because fabricengine already keeps weft scratch untracked through `info/exclude` and a committed pattern would be a second mechanism for the same rule.
- Board sync keeps its stage-all rather than staging a list of board paths, because the board writes many file kinds and such a list would drift from them.

## Verification

`lym gate test --tier 1` over `internal/fabricengine`, `internal/fabriccli` and `internal/boardengine` is green.
Five back-to-back runs of `internal/fabriccli`'s tier1 tests wrote no `fabriccli.test` trace into the live hub's `_board/.lym/logs`;
the traces that still appear there come from concurrent gate runs of branches that do not carry this change yet.
Tier0 over the diff scope is green on a tree that carries main's markdown-link fix for the bench data under `docs/research/`.
