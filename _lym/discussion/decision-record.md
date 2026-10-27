# Decision record — test-logs-leak-into-bolt

## Goal

No `go test` run writes trace logs into the live hub, and no machine-local file under a `.lym/` directory is ever committed to Bolt by `board sync`.

## Scope

- The `fabriccli` tier1 hub-mutation scenario's bypass-push step, which runs `fabriccli.RunCLI` with the process cwd.
  The bypass branch resolves the hub from cwd and calls `logger.SetDurableSinkDir(fabricengine.HubLogsDir(hub))`;
  under `go test` the cwd is the package directory inside the real hub's task worktree, so the redirect lands on the live `_board/.lym/logs`, and every later record of that test binary follows it.
- `fabricengine.Bolt.Commit`, the stage-all commit `boardengine.Sync` runs on `_board`.
  It seeds no git excludes, so a board whose weft common gitdir lacks the `.lym/` exclude commits whatever lies under `_board/.lym/`.

Out of scope: the logger's override semantics, the bypass branch's production resolution, and `boardengine`'s own `.gitignore` seeding.

## Decisions

### D1: The bypass-push step runs from the test hub's board worktree

Decision: the step calls `fabriccli.RunCLIIn(h.BoardDir(), …)` and restores the process-wide trace sink with `logger.SetDurableSinkDir("")` on cleanup.
Rationale: the detached push child starts in the hub's board worktree, so this is the production cwd, and it resolves the test's own hub;
the cleanup keeps the redirect from outliving the step in the shared test binary.

### D2: `Bolt.Commit` seeds the weft artifact excludes before its stage-all

Decision: `Bolt.Commit` calls `seedWeftArtifactExcludes` on its repo before staging, so `.lym/`, the lock files and `.weft/` are excluded on every board sync.
Rationale: the exclude file is fabricengine's sole-owned mechanism for keeping weft scratch untracked, and today only clone, reconcile and the weft verbs seed it;
seeding at the stage-all makes the board self-healing whatever seeded the gitdir before.
Bound: the seed only appends exclude lines already owned by `seedWeftArtifactExcludes`; it adds no pattern and stages nothing.
A seed failure fails the commit, since committing without the excludes is the defect.

### Added after Discussion (parent, 2026-10-27): Leave the bench link breaks to main

Decision: Do not touch the docs/research bench files or the docs link allowlist in this run. Task docs-link-skip-bench-data fixes TestEnforcement_MarkdownLinks on main by skipping docs/research/*/data/; once it lands, the orch resumes this run and the run takes main through its normal Parent-Align.

Rationale: The breaks come from main's bench commits, not this change; one fix on main serves every run.

## Constraints

- `PATTERN-fabric-git`: the exclude seeding stays inside `internal/fabricengine`.
- `PATTERN-durable-vs-ephemeral-state`: `.lym/` is never tracked; the pattern comes from `lymdirs.DotLymDirName`.
- `PATTERN-test-isolation`: the scenario keeps its `hubforge` fixture.

## Auto-mode assumptions

- The leaking binary was the `fabriccli` test binary: the committed trace headers name `fabriccli.test` with an empty `worktree_root`, the shape `SetDurableSinkDir` leaves, and its records carry `TestRunCLI_HubMutationScenario` paths.
- Seeding the exclude is preferred over a committed `_board/.gitignore` entry, because fabricengine's doc names info/exclude as the weft mechanism for `.lym/`.

## Open risks

- Another test that runs a hub-resolving verb with the process cwd could leak again; D2 keeps any such file out of Bolt.

## Acceptance criteria

- `Bolt.Commit` on a repo whose gitdir has no excludes commits a dirty board file and leaves a file under `.lym/` untracked.
- The hub-mutation scenario's bypass push still advances both bare upstreams, running from the test hub's board worktree.
- `lym gate test` is green over the touched packages.
