# Decision record — lyx creates hub state under a Claude Code agent worktree

## Goal

A `lyx` process started in a git worktree that is not a worktree of a hub, such as a Claude Code agent worktree under `<prime>/.claude/worktrees/`, creates no hub state under the guessed `<parent>/_board/.lyx`.

## Scope

- `tendercli.InstallNetworkGate`, the root pre-run's installer of the process-wide pause gate, whose in-flight markers created `<prime>/.claude/worktrees/_board/.lyx/pause/net`.
- Its integration test in `internal/tendercli/cli_test.go` and the tendercli package doc.
- Out of scope: resolving the real hub from an agent worktree's git common directory, and every other hub-state writer, which already refuses or gates on hub presence.

## Decisions

### The gate installs only where the resolved hub has a board-level `_lyx`

Decision: `InstallNetworkGate` installs nothing unless `preflight.BoardLyxPresent` reports the resolved location's board-level `_lyx` present.
Rationale: `lyxcwd.Resolve` takes the parent of any git worktree root as the hub, so it succeeds in a plain repository or an agent worktree nested in the prime;
the board-level `_lyx` is the predicate hub reconcile already uses to tell a hub from a plain repository.
Bound: a process outside a hub is held by no pause, which matches today's behavior, since the record at a guessed path never existed and so never held a call.

## Constraints

- `PATTERN-cwd-resolution`: the hub is still resolved only through `lyxcwd`.
- `PATTERN-told-geometry`: the hub check goes through `preflight`'s existing predicate, not a new path computation.
- `PATTERN-test-economy`: the agent-worktree case extends the existing gate test rather than adding a test.

## Auto-mode assumptions

- Refusing, in the form of installing no gate, satisfies the board entry's "resolves the board dir from the fabric anchor or refuses".

## Open risks

- A hub whose `_board/_lyx` is damaged runs without the gate, as hub reconcile already skips it.

## Acceptance criteria

- From a linked git worktree at `<prime>/.claude/worktrees/<name>`, `InstallNetworkGate` installs no gate, and a network call creates no `<prime>/.claude/worktrees/_board`.
- From outside any git repository, it still installs nothing.
- From a pair's code worktree, it still installs the gate with the pair's slug.
