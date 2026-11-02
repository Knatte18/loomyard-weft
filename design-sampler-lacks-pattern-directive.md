# The sampler agent gets no PATTERN directive

`sampler-lacks-pattern-directive` · Note · bug, loom, stencil, pattern

## Seen (operator, 2026-11-02)

An interactive sampler agent told the operator that PATTERN reaches only the loom agents, not an interactive session: working directly with Claude in the code, it does not see the abscissa rule.

## Cause (orch, from the source)

`internal/samplerengine` and `contracts/stencils/sampler/sampler-template-agent.md` name no PATTERN directive, and the stencil's markers are `slug`, `task_worktree`, `decision_record_path`, `parent_directive` and `edit_directive` only.
Every other code-writing role renders `pattern.Directive` into its prompt: Webster (`internal/websterengine`), darn (`internal/darnengine/spec.go`), Drift-Verify (`internal/driftengine/seats.go`), the Discussion and Plan writers (`internal/loomengine`) and the review seats (`internal/burlerengine`).
The sampler is the one role that writes and commits code and gets none.

## Fix

Render the PATTERN directive into the sampler's prompt with `pattern.Directive`, as `VerifyTable` does for its implementer seat, and add a marker for it to the stencil; the role is `pattern.RoleImplementer` while the sampler writes code.
Add a test that the sampler's prompt carries the directive.
If the sampler becomes a read-only discussion role (see note `discussed-task-split`), it needs the directive too, since what it proposes must not break PATTERN; the role then follows the Discussion writer's.
Bundle with the other small sampler changes when the operator has settled the sampler's direction; do not offer this note alone as a task.
