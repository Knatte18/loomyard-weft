## Goal

```markdown
## Seen (orch, 2026-11-02, from the darn writer of `tmuxkit-short-socket-path`)

With the load average about 30 and two or three tier1 runs sharing the machine, `internal/fabricengine` tier1 fails with `panic: test timed out after 10m`.
The same timeout hit `beam-attach-no-orch` (gate log `20261102T121452Z-tier1-3498956`) and `explore-subagent-model` (`20261102T122043Z-tier1-3680568`) before their branches had edits, so the cause is not the change under test.
The package takes about 349 s alone on this machine (`test-time-audit-3`), so 10m leaves little room for load.

## Idea

The gate passes no `-timeout`, so go test runs its default 10m.
Set a `-timeout` for tier1 in the gate (config, not a literal), or cut the package time with the `/dev/shm` path of task `tmuxkit-short-socket-path`.
The writer decides after that task has measured the package under `/dev/shm`.

## Update (orch, 2026-11-02)

`tmuxkit-short-socket-path` has measured `internal/fabricengine` tier1 under `/dev/shm`: it passes but is only 2% to 7% faster, so the `/dev/shm` route does not cut the package time.
The fix is the timeout: raise the `-timeout` of the tier1 `go test` run through the gate, as a config key beside the gate's other settings with a default above the package's wall under load (about 600 s seen), not a literal.
The writer first reads how the gate builds its `go test` command line and confirms from the code whether it sets `-timeout` at all, since the 10m in the panic is `go test`'s own default and this note only assumes the gate passes none.
The cause of the package's slowness is not part of this task.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-02): The hub gate command switches to the placeholder after landing

Decision: The change lands with the {test_timeout} mechanism only; the hub's tier1 gate.yaml keeps its GOFLAGS -timeout=20m stopgap, and the parent edits tiers.tier1.command to carry -timeout {test_timeout} and drops the GOFLAGS flag once the landing's deploy has installed the lym binary that knows the placeholder.

Rationale: The current binary would pass the placeholder through literally, and the hub's gate.yaml lies outside the task's write surface; the stopgap keeps verify green meanwhile.

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - tier1-command-lacks-test-timeout (class decision, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
