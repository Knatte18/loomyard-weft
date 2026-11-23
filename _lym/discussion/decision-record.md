## Goal

```markdown
Imported from [issue #186](https://github.com/Knatte18/loomyard/issues/186).

Seen: on a loom run, the recovery agent and the code fixer each mailed the parent orch a question and ended their session before the reply arrived; the reply came back from creel as returned. The drift judge instead put its question in its own pane, where no notice reaches the orch, and the run sat waiting until the operator noticed.

Workaround: the orch watched the run's idle panes and answered by creel mail; the returned replies were dropped, and the runs went on without them.

Expected: an agent that asks with --expect-reply waits for the reply before it ends, and every run agent, the drift judge included, asks its parent by creel mail rather than in its pane.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-23): Accept round 1: F1 fixed, F2 fixed differently

Decision: Accept the code review at round 1. BLOCKING F1 (probe-refused-from-nonorch-strand-pane) is fixed. BLOCKING F2 (quiet-probe-release-ends-run-mid-turn) is fixed differently: the release keeps the two quiet probes and also requires the agent's pane to read idle, as startHoldReleased reads it, so a turn the reply wakes keeps the hold while a settled question still releases. F3 to F9 are fixed, F8 through an exported Runner.HasReplyProbe accessor.

Rationale: F2's own change is stricter than the proposal without holding a settled question to its deadline; every fix is committed with its gates passing.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
