# Keep gate-test trace logs out of the live hub and out of Bolt

A `go test` run of `fabriccli` wrote `trace-*.log` files into the live hub's `_board/.lym/logs`, and the next `board sync` committed them to Bolt.
Two defects combined, and this change fixes both.

## What changes

- The `fabriccli` hub-mutation scenario's bypass-push step now runs `fabriccli.RunCLIIn(h.BoardDir(), …)` instead of `RunCLI` with the process cwd.
  The bypass branch resolves the hub from cwd and calls `logger.SetDurableSinkDir(fabricengine.HubLogsDir(hub))`.
  Under `go test` the cwd is the package directory inside the real hub's task worktree, so the trace sink moved to the live `_board/.lym/logs` and every later record of that test binary followed it.
  The step now resolves the test's own `hubforge` hub, which is also the cwd the detached push child has in production.
  Its cleanup resets the process-wide sink with `logger.SetDurableSinkDir("")`, so the redirect does not outlive the step.
- `commitWeftAt`, and so `Bolt.Commit` and the stage-all commit of `boardengine.Sync`, now calls `seedWeftArtifactExcludes` before staging.
  A board whose weft common gitdir lacks the `.lym/` exclude no longer commits whatever lies under `_board/.lym/`.
  Any other stray machine-local file there stays out of Bolt too.
- The comment in `CloneHub` that called the board's excludes not self-healing is rewritten, since the commit now seeds them itself.
  `internal/fabricengine/doc.go` records the new behavior.

## Design decisions

- The exclude is seeded at the stage-all, not committed as a `_board/.gitignore` entry.
  The info/exclude file is fabricengine's sole-owned mechanism for keeping weft scratch untracked, and the seed adds no pattern beyond those `seedWeftArtifactExcludes` already owns.
- A seed failure fails the commit before anything is staged, because committing without the excludes is the defect.
- The `CloneHub` seed stays, so `_board/.lym` never shows as untracked dirt from the hub's birth on.
- Out of scope: the logger's override semantics, the bypass branch's production resolution and `boardengine`'s own `.gitignore` seeding.

## Tests

- `commitweftat_test.go` covers `commitWeftAt` on a repo whose gitdir has no excludes: a dirty board file is committed, a file under `.lym/` stays untracked, and a seed failure fails the call before staging.
- `bolt_integration_test.go` is adjusted to the new behavior.
- The hub-mutation scenario still asserts that the bypass push advances both bare upstreams.

## Check by hand

- `lym gate test` over `./internal/fabricengine` and `./internal/fabriccli` was not run while writing this description.
- Another test that runs a hub-resolving verb with the process cwd could still redirect the sink into the live hub.
  The seed keeps the resulting files out of Bolt, but a leak into `_board/.lym/logs` would no longer show in `board sync`.
- `TestEnforcement_MarkdownLinks` may fail on the docs/research bench files.
  Those breaks come from main's bench commits, and task `docs-link-skip-bench-data` fixes them on main.
